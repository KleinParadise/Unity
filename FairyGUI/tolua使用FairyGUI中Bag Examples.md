1. 打开FairyGUI-unity-master/LuaSupport/README.md文件,参照步骤2,步骤3执行
2. 在当前工程Assets/LuaFramework/Lua文件夹中新建Bag文件夹。
3. 在Assets/LuaFramework/Lua/Bag文件夹新建BagMain.lua
```lua
require "Bag/BagWindow"
BagMain = {}
--需要在CustomSettings.cs的customDelegateList加入	
--_DT(typeof(EventCallback0)),_DT(typeof(EventCallback1)),否则bagBtn监听事件不生效
function BagMain:Start()
	--获取UIPanel GameObject
	local UIPanelGo = UnityEngine.GameObject.Find("UIPanel")
	if UIPanelGo then
		--获取UIPanel组件
		local UIPanel = UIPanelGo:GetComponent("UIPanel")
		if UIPanel then
			--获得FairyGUI 当前view
			logWarn('BagMain:Start--->>>'..tostring(UIPanel));
			local mainView = UIPanel.ui
			--现在bagwindow实例
			local bagWin = BagWindow.New()
			local function OnClick(content)
				bagWin:Show()
			end
			--从view获取按钮,并设置点击事件 
			local bagBtn = mainView:GetChild("bagBtn")
			bagBtn.onClick:Add(OnClick)
		end
	end
end
```
3. 在Assets/LuaFramework/Lua/Bag文件夹新建BagWindow.lua
```lua
require "FairyGUI"
BagWindow = fgui.window_class()
local contentPane

function BagWindow:clickItem(context)
	local item = context.data
	contentPane:GetChild("n11").asLoader.url = item.icon
	contentPane:GetChild("n13").text = item.icon
end

local function RenderListItem(index,button)
   logWarn('RenderListItem -->>>');
	button.icon = "i" .. math.random(0, 9)
	button.title = "" .. math.random(0, 100)
end

function BagWindow:ctor()
    print('MyWinClass-ctor')
    self.contentPane = UIPackage.CreateObject("Bag", "BagWin")
    contentPane = self.contentPane
    self:Center()
	self.modal = true

	local _list = self.contentPane:GetChild("list").asList
	_list.onClickItem:Add(BagWindow.clickItem,self)
	_list.itemRenderer = RenderListItem
	_list.numItems = 45
end

function BagWindow:DoShowAnimation()
	self:SetScale(0.1, 0.1)
	self:SetPivot(0.5, 0.5)
	self:TweenScale(Vector2.New(1, 1), 0.3):OnComplete(self.OnShown)
end

function BagWindow:DoHideAnimation()
	self:TweenScale(Vector2.New(0.1, 0.1), 0.3):OnComplete(self.HideImmediately)
end

function BagWindow:OnShown()
    print('MyWinClass-onShown')
end
```

4. 修改GLoader.cs文件使其支持从lua继承 主要加了#if FAIRYGUI_TOLUA #endif 的内容
```c#
using UnityEngine;
using FairyGUI.Utils;
#if FAIRYGUI_TOLUA
using LuaInterface;
#endif

namespace FairyGUI
{
		override public void Dispose()
		{
			if (_content.texture != null)
			{
				if (_contentItem == null)
					FreeExternal(image.texture);
			}
			if (_errorSign != null)
				_errorSign.Dispose();
			if (_content2 != null)
				_content2.Dispose();
			_content.Dispose();
			base.Dispose();
#if FAIRYGUI_TOLUA
			if (_peerTable != null)
			{
				_peerTable.Dispose();
				_peerTable = null;
			}
#endif
		}

		virtual protected void LoadExternal()
		{
			Texture2D tex = (Texture2D)Resources.Load(_url, typeof(Texture2D));
			if (tex != null)
				onExternalLoadSuccess(new NTexture(tex));
			else
				onExternalLoadFailed();
#if FAIRYGUI_TOLUA
			CallLua("LoadExternal");
#endif
		}

		override public void Setup_BeforeAdd(ByteBuffer buffer, int beginPos)
		{
			base.Setup_BeforeAdd(buffer, beginPos);

			buffer.Seek(beginPos, 5);

			_url = buffer.ReadS();
			_align = (AlignType)buffer.ReadByte();
			_verticalAlign = (VertAlignType)buffer.ReadByte();
			_fill = (FillType)buffer.ReadByte();
			_shrinkOnly = buffer.ReadBool();
			_autoSize = buffer.ReadBool();
			showErrorSign = buffer.ReadBool();
			_content.playing = buffer.ReadBool();
			_content.frame = buffer.ReadInt();

			if (buffer.ReadBool())
				_content.color = buffer.ReadColor();
			_content.fillMethod = (FillMethod)buffer.ReadByte();
			if (_content.fillMethod != FillMethod.None)
			{
				_content.fillOrigin = buffer.ReadByte();
				_content.fillClockwise = buffer.ReadBool();
				_content.fillAmount = buffer.ReadFloat();
			}

			if (!string.IsNullOrEmpty(_url))
				LoadContent();
		}
#if FAIRYGUI_TOLUA
			internal LuaTable _peerTable;

			public void SetLuaPeer(LuaTable peerTable)
			{
				_peerTable = peerTable;
			}

			[NoToLua]
			public bool CallLua(string funcName)
			{
				if (_peerTable != null)
				{
					LuaFunction ctor = _peerTable.GetLuaFunction(funcName);
					if (ctor != null)
					{
						ctor.Call(this);
						ctor.Dispose();
						return true;
					}
				}

				return false;
}
#endif
	}
}
```

5. 修改UIObjectFactory.cs文件,增加SetLuaExtension函数,使其支持回调lua代理函数
```c#
#if FAIRYGUI_TOLUA
public static void SetLuaExtension(System.Type baseType, LuaFunction extendFunction)
{
	SetLoaderExtension(() =>
		{
			GLoader gcom = (GLoader)Activator.CreateInstance(baseType);
			extendFunction.BeginPCall();
			extendFunction.Push(gcom);
			extendFunction.PCall();
			gcom.SetLuaPeer(extendFunction.CheckLuaTable());
			extendFunction.EndPCall();

			return gcom;
		});
}
#endif
```
6. 完成4,5两步即在lua可以写类继承GLoader。在Assets/LuaFramework/Lua/Bag文件夹新建MyGLoader.lua
```lua
require "FairyGUI"
MyGLoader= fgui.extension_class(GLoader)
function MyGLoader:LoadExternal()
	logWarn('MyGLoader:LoadExternal--->>> enter'..tostring(self.url));
	local inst = IconManager.inst
	inst:LoadIcon(self.url,
		function(texture) 
			self:onExternalLoadSuccess(texture)
		end,
		function(error) 
			self:onExternalLoadFailed()
		end)

end
```

7. 修改Assets/LuaFramework/Lua/Logic/Game.lua中Game.OnInitOK()函数
```lua
function Game.OnInitOK() 
    logWarn('LuaFramework InitOK--->>>');
    FairyGUI.UIObjectFactory.SetLuaExtension(typeof(MyGLoader.base), MyGLoader.Extend)
    BagMain:Start()
end
```

8. 修改CustomSettings.cs文件,在customDelegateList新增
```c#
_DT(typeof(EventCallback0)),
_DT(typeof(EventCallback1)),
_DT(typeof(LoadCompleteCallback)), 
```
在customTypeList新增
```c#
_GT(typeof(IconManager)),
_GT(typeof(UIPanel)),
```

9. 执行Lua/Gen Lua Delegates 和 Lua/Gen LuaWrap + Binder命令生成wrap文件

10. 参考集成FairyGUI到Unity工程.md,拷贝对应的资源文件,场景中新建UIPanelGameObject并设置正确的包名(Package Name)和组件名(Component Name)

11. 启动游戏,集成成功




