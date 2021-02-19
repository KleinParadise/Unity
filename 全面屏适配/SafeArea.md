适配思路:获得刘海的大小 ---> 计算出显示框Rect --> Rect与实际屏幕大小比例 ---> 设置View的RectTransform的AnchorMin与AnchorMax (RectTransform的AnchorMin与AnchorMax 设置View框的大小)  

```lua
--显示框Rect
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

--Rect与实际屏幕大小比例
function CalcuAnchor(self)
  local r = self:GetSafeArea()
  local anchorMin = r.position
  local anchorMax = r.position + r.size
  anchorMin.x = anchorMin.x /  ScreenResizeHelper.width
  anchorMin.y = anchorMin.y /  ScreenResizeHelper.height
  anchorMax.x = anchorMax.x /  ScreenResizeHelper.width
  anchorMax.y = anchorMax.y /  ScreenResizeHelper.height
  return anchorMin,anchorMax
end

--设置View的RectTransform的AnchorMin与AnchorMax
function ApplySafeArea(self,transform)
  local anchorMin,anchorMax = self:CalcuAnchor()
  transform.anchorMin = anchorMin
  transform.anchorMax = anchorMax
end
```

unity中获取安卓手机屏幕旋转角度

1. 在MainActivity中声明成员变量 
```java 
  private OrientationEventListener mOrientationListener; //(OrientationEventListener系统屏幕旋转回调,improt android.view.OrientationEventListener)
  private String mCurrentDir;
```


2. 在MainActivity的onResume函数中初始化旋转回调
```java
@Override
protected void onResume() {
    super.onResume();

    // 注册方向回调，检测屏幕方向改变
    if (null == mOrientationListener) {
        mOrientationListener = new OrientationEventListener(this) {
            @Override
            public void onOrientationChanged(int orientation) {
               //这里只考虑左横屏和右横屏的情况
               if(orientation == OrientationEventListener.ORIENTATION_UNKNOWN){
                  return ;
               }
               String dir = "";
               if(orientation > 80 && orientation < 100){
                  dir = "right";
               }else if(orientation > 260 && orientation < 280){
                   dir = "left";
               }
               if(dir != "" && dir != mCurrentDir){
                  UnityUtils.SendLuaEvent();//通知lua方向改变做业务处理
                  mCurrentDir = dir;
               }
            }
        };
        mOrientationListener.enable();
    }
}
```

3. 在MainActivity的onPause函数中销毁旋转回调
```java
 @Override
protected void onPause() {
    super.onPause();

    if (null != mOrientationListener) {
        mOrientationListener.disable();
        mOrientationListener = null;
    }
}
```
