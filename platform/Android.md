### Unity与Android交互

1. 游戏Android工程基本都会有一个UnityXXXXActivity的主Activity,它需要继承UnityPlayerActivity。当unity程序启动就会调用该UnityXXXXActivity。(游戏启动调用哪个activity,是在AndroidManifest.xml中配置的)

2. 通过AndroidJavaClass和AndroidJavaObject两个工具类来实现从unity调用到Android java的函数
```c#
 // UnityPlayer是FrameLayout的一个子类，而currentActivity则是UnityPlayer类中的静态对象。
 AndroidJavaClass jc = new AndroidJavaClass("com.unity3d.player.UnityPlayer");
 // 通过GetStatic获取了静态变量currentActivity的值，它的类型是AndroidJavaObject（Unity3d的JavaObject的基本类型）。
 AndroidJavaObject jo = jc.GetStatic<AndroidJavaObject>("currentActivity");
 jo.Call("currentActivity中函数","参数");
 //UnityPlayer就是一个Unity3d生成的一个类，我们通过new（”完整类名”）的方式获取到了这个类，然后通过getStatic获取到了currentActivity这个静态对象。
 //currentActivity这个类如何找到调用方法？
 //UnityXXXXActivity<-- UnityPlayerActivity <--UnityPlayerNativeActivity <-- NativeActivity<-- Activity(各个Activity的继承关系)
 //在 UnityPlayerNativeActivity 这个类中，声明了一个UnityPlayer对象
 //UnityPlayerNativeActivityoncreate方法里面初始化了这个对象，把UnityPlayerNativeActivity上下文作为参数传了进去
 this.mUnityPlayer = new UnityPlayer(this)
 //在UnityPlayer类的构造方法中，把当前的上下文赋值给了currentActivity
 currentActivity = (Activity)paramContextWapper
 //UnityXXXXActivity重写了UnityPlayerNativeActivity的onCreate方法，导致UnityPlayer构造器传入的参数是UnityXXXXActivity的实例
```

3. 通过UnityPlayer.UnitySendMessage函数来实现Android java调用到unity的函数
```java
UnityPlayer.UnitySendMessage(gameobjectname,funcname,argus);
//参数1 为游戏中脚本gameobject
//参数2 为该脚本中的函数
//参数3 调用函数的参数
```
