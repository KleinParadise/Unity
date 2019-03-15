### Hello World
1. 打开unity新建工程UnityToLuaProject。新建完成后,将UnityToLuaProject工程文件夹里面的内容全部删除。
2. 解压下载的LuaFramework_UGUI-master.zip。将解压后LuaFramework_UGUI-master文件夹中的所有文件全部拷贝至UnityToLuaProject工程文件夹。
3. 重新打开unity,如Console无报错,则导入成功。
4. 在unity导航栏中选择tolua框架生成的LuaFramework页签,在下拉菜单中点击Bulid windows resourse,将lua代码打包为AssetBundle。
5. 在unity导航栏中选择tolua框架生成的Lua页签,在下拉菜单中点击Grnerate all,生成c#与lua交互的warp文件。
6. 双击Assets/LuaFramework/Scene/Main场景,点击运行按钮,框架Hello World启动运行。

### ToLua框架中各个文件夹的作用。
![Image of unityfile](https://github.com/KleinParadise/Unity/blob/master/pic/unity_tolua_file.png)
1. Editor：里面是例子用到的一个新手引导步骤演示的编辑器脚本。
2. Examples ：框架自带的Demo例子
3. Lua：框架自带的Lua源码目录，用户自定义的Lua脚本也就是放在这里面，最后打包的时候，打包脚本会将其按目录结构生成到StreaminAssets目录里面去，然后在将其上传到游戏的Web服务器上面，用于准备被每个游戏客户端下载更新他们本地的Lua脚本。达到热更目的。
   1. 3rd：里面是第三方的一些插件lua、实例源码文件，比如：cjson、pbc、pblua、sproto等。
   2. Common：公用的lua文件目录，如define.lua文件，一些变量声明，全局配置等，functions.lua常用函数库，通讯的protocal.lua协议文件。
   3. Controller：控制器目录，它不依赖于某一个Lua面板，它是独立存活在Luavm中的一个操作类，操作数据、控制面板显示而已。
   4. Logic：目录里面存放的是一些管理器类，比如GameManager游戏管理器、NetworkManager网络管理器，如果你有新的管理器可以放到里面。
   5. View：这是面板的视图层，里面都是一些被Unity调用的面板的变量，走的是Unity GameObject的生命周期的事件调用。
4. Scenes：Hello World 开始场景
5. Scripts：Hello World 运行需要的c#文件,以及框架封装的游戏各模块的Manager,网络库,热更新,对象池,工具类等c#文件
6. ToLua: lua与c#的交互文件,lua的工具类

### 利用ToLua框架把编写的lua代码打包成AssetBundle
1. 在hello world第4步，当按下LuaFramework-Build XXX Resources的时候，框架会自动将Assets/LuaFramework/Lua下的所有内容打成AssetBundle包，放在StreamingAssets下。点击按钮等待运行完成，StreamingAssets文件夹下会有个Lua文件夹，里面放的就是Assets/LuaFramework/Lua在打包之后的结果。  
2. StreamingAssets/Lua文件夹下，除了资源包外，还有一个3rd文件夹，可以打开Assets/LuaFramework/Lua/3rd 目录，然后打开其中一个，例如cjson的文件夹，可以发现，这文件夹里除了一些lua文件，还有一些txt配置文件或说明文件，所以，这个StreamingAssets/Lua/3rd文件夹下，放的就是这些lua文件外的文件资源。  
3. 如果我们要打包一个自定义的Lua文件（不是框架提供的Main.lua文件）的话，那么我们完全可以先在Assets/LuaFramework/Lua这个文件夹下，自定义一个专门存放我们编写的Lua文件的文件夹，当打包出来后会发现，StreamingAssets/Lua下会有一个自定义的文件夹名.unity3d的一组资源包。  
4. 不管是不是自己定义lua入口文件，最好都放在Assets/LuaFramework/Lua下，不要再另外加文件夹，因为放在Assets/LuaFramework/Lua路径下的所有lua文件都会直接一同打包在lua.unity3d资源包中，而程序设置好就是去这个包里读取lua的入口文件，除非你会改动lua入口文件的读取路径。
5. 如果想自定义lua入口,不用框架提供给我们的Main.lua这个文件：找到LuaManger这个类，然后找到StartMain()这个方法。
```c#
void StartMain() {
   lua.DoFile("Main.lua");
   LuaFunction main = lua.GetFunction("Main");
   main.Call();
   main.Dispose();
   main = null;    
}
```
只需要把lua.DoFile里面的参数修改为你自定义的lua文件名，然后LuaFunction main=lua.GetFunction(“Main”)这一行的括号内的参数，修改为作为入口且存在于你自定义的lua文件中的lua方法即可。

### 利用ToLua框架把资源打包成AssetBundle
1. 
