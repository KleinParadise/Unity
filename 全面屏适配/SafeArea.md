适配思路:获得刘海的大小 ---> 计算出显示框Rect --> Rect与实际屏幕大小比例 ---> 设置View的RectTransform的AnchorMin与AnchorMax (RectTransform的AnchorMin与AnchorMax 设置View框的大小)  

```lua
function GetSafeArea(self)
  local safeArea
  local width = ScreenResizeHelper.width --c#获取手机的实际宽度
  local height = ScreenResizeHelper.height --c#获取手机的实际高度
  if self.iBarHeight > 0 then --获取的刘海屏高度
      if self.sDriection == "right" then
        safeArea = Rect(5,0,width - self.iBarHeight - 5,height)
      else
        safeArea = Rect(self.iBarHeight + 5,0,width,height)
      end
  end
  return safeArea
end
```
