### toLua中wrap文件的使用流程
1. 在CustomSettings.cs的customTypeList添加要导入到Lua使用的类
2. 通过ToLuaMenu.cs的GenerateClassWraps()函数生成每个需要导入类的warp文件
3. 通过ToLuaMenu.cs的GenLuaBinder()函数生成调用每个warp文件的Register()函数的函数。该函数生成在LuaBinder.cs的Bind()函数中。
4. 开启lua虚拟机时,通过LuaBinder.cs的Bind函数将类导入到虚拟机中。

### 单个warp文件的生成流程
```c#
//通过ToLuaMenu.cs的GenerateClassWraps函数生成所有需要导入到Lua使用的类的warp文件
//将className，type等信息放在ToLuaExport中
ToLuaExport.Clear();
ToLuaExport.className = list[i].name;
ToLuaExport.type = list[i].type;
ToLuaExport.isStaticClass = list[i].IsStatic;            
ToLuaExport.baseType = list[i].baseType;
ToLuaExport.wrapClassName = list[i].wrapName;
ToLuaExport.libClassName = list[i].libName;
ToLuaExport.extendList = list[i].extendList;
ToLuaExport.Generate(CustomSettings.saveDir);

//通过ToLuaExport的Generate()函数生成wrap文件
public static void Generate(string dir)
{
#if !EXPORT_INTERFACE
    Type iterType = typeof(System.Collections.IEnumerator);

    if (type.IsInterface && type != iterType)
    {
        return;
    }
#endif

    //Debugger.Log("Begin Generate lua Wrap for class {0}", className);        
    sb = new StringBuilder();
    usingList.Add("System");

    if (wrapClassName == "")
    {
        wrapClassName = className;
    }

    if (type.IsEnum)
    {
        BeginCodeGen();
        GenEnum();                                    
        EndCodeGen(dir);
        return;
    }
    //生成函数信息列表包括成员函数,get/set函数
    InitMethods();
    //生成属性信息列表
    InitPropertyList();
    //生成构造函数列表
    InitCtorList();
    //生成类名
    BeginCodeGen();
    //生成向lua注册的绑定函数
    GenRegisterFunction();
    //生成构造绑定函数
    GenConstructFunction();
    //生成this[]绑定函数
    GenItemPropertyFunction();
    //生成其他函数
    GenFunctions();
    //GenToStringFunction();
    //生成每个成员变量的get函数
    GenIndexFunc();
    //生成每个成员变量的set函数
    GenNewIndexFunc();
    注册带out参数的函数,out参数作为额外的参数返回给lua
    GenOutFunction();
    //生成类中的委托绑定函数
    GenEventFunctions();
    //保存文件
    EndCodeGen(dir);
}
```

### 单个wrap文件的解析
#### 注册部分
```c#
public static void Register(LuaState L)
{
    L.BeginClass(typeof(UnityEngine.GameObject), typeof(UnityEngine.Object));
    L.RegFunction("CreatePrimitive", CreatePrimitive);
    L.RegFunction("GetComponent", GetComponent);
    L.RegFunction("GetComponentInChildren", GetComponentInChildren);
    L.RegFunction("GetComponentInParent", GetComponentInParent);
    L.RegFunction("GetComponents", GetComponents);
    L.RegFunction("GetComponentsInChildren", GetComponentsInChildren);
    L.RegFunction("GetComponentsInParent", GetComponentsInParent);
    L.RegFunction("SetActive", SetActive);
    L.RegFunction("CompareTag", CompareTag);
    L.RegFunction("FindGameObjectWithTag", FindGameObjectWithTag);
    L.RegFunction("FindWithTag", FindWithTag);
    L.RegFunction("FindGameObjectsWithTag", FindGameObjectsWithTag);
    L.RegFunction("Find", Find);
    L.RegFunction("AddComponent", AddComponent);
    L.RegFunction("BroadcastMessage", BroadcastMessage);
    L.RegFunction("SendMessageUpwards", SendMessageUpwards);
    L.RegFunction("SendMessage", SendMessage);
    L.RegFunction("New", _CreateUnityEngine_GameObject);
    L.RegFunction("__eq", op_Equality);
    L.RegFunction("__tostring", ToLua.op_ToString);
    L.RegVar("transform", get_transform, null);
    L.RegVar("layer", get_layer, set_layer);
    L.RegVar("activeSelf", get_activeSelf, null);
    L.RegVar("activeInHierarchy", get_activeInHierarchy, null);
    L.RegVar("isStatic", get_isStatic, set_isStatic);
    L.RegVar("tag", get_tag, set_tag);
    L.RegVar("scene", get_scene, null);
    L.RegVar("gameObject", get_gameObject, null);
    L.EndClass();
}
```
上述代码通过GenRegisterFunction()函数完成。分为4个部分：
1. BeginClass负责类在lua中初始化部分
2. RegFunction部分，负责将函数注册到lua中
3. RegVar部分，负责将变量和属性注册到lua中
4. EndClass部分，负责类结束注册的收尾工作

##### BeginClass
①用于创建类和类的元表,类的元表的元表（类的元表是承载每个类方法和属性的实体，类的元表的元表就是类的父类）  
②将类添加到loaded表中。  
③设置每个类的元表的通用的元方法和属性，__gc,name,ref,__cal,__index,__newindex。  

##### RegFunction
将每个函数转化为一个指针，然后添加到类的元表中去，与将一个c函数注册到lua中是一样的。

##### RegVar
每一个变量或属性或被包装成get_xxx,set_xxx函数注册添加到类的元表的gettag，settag表中去，用于调用和获取

##### EndClass
①设置类的元表  
②把该类加到所在模块代表的表中（如将GameObject加入到UnityEngine表中）

#### 函数实体部分
```c#
[MonoPInvokeCallbackAttribute(typeof(LuaCSFunction))]
static int GetComponent(IntPtr L)
{
    try
    {
        //获取栈中参数的个数
        int count = LuaDLL.lua_gettop(L);
        //根据栈中元素的个数和元素的类型判断该使用那一个重载
        if (count == 2 && TypeChecker.CheckTypes<string>(L, 2))
        {
            //将栈底的元素取出来，这个obj在栈中是一个fulluserdata，需要先将这个fulluserdata转化成对应的c#实例，也就是调用这个GetComponent函数的GameObject实例
            UnityEngine.GameObject obj = (UnityEngine.GameObject)ToLua.CheckObject(L, 1, typeof(UnityEngine.GameObject));
            //将栈底的上一个元素取出来，也就是GetComponent(string type)的参数
            string arg0 = ToLua.ToString(L, 2);
            //通过obj，arg0直接第调用GetCompent(string type)函数
            UnityEngine.Component o = obj.GetComponent(arg0);
            //将调用结果压栈
            ToLua.Push(L, o);
            //返回参数的个数
            return 1;
        }
        //另一个GetComponent的重载，跟上一个差不多，就不详细说明了
        else if (count == 2 && TypeChecker.CheckTypes<System.Type>(L, 2))
        {
            UnityEngine.GameObject obj = (UnityEngine.GameObject)ToLua.CheckObject(L, 1, typeof(UnityEngine.GameObject));
            System.Type arg0 = (System.Type)ToLua.ToObject(L, 2);
            UnityEngine.Component o = obj.GetComponent(arg0);
            ToLua.Push(L, o);
            return 1;
        }
        //参数数量或类型不对，没有找到对应的重载，抛出错误
        else
        {
            return LuaDLL.luaL_throw(L, "invalid arguments to method: UnityEngine.GameObject.GetComponent");
        }
    }
    catch (Exception e)
    {
        return LuaDLL.toluaL_exception(L, e);
    }
}
```

#### lua调用c#函数的过程
```lua
local tempGameObject = UnityEngine.GameObject("temp")
local transform = tempGameObject.GetComponent("Transform")
```
第二句代码的实际调用过程:  
1. 从tempGameObject的元表查找GetComponent函数。
2. 取到了GetComponent函数后,将tempGameObject，"Transform"作为参数压入桟中,然后调用GetComponent函数。
3. 进入到GetComponent内部操作,因为生成了新的ci，所以此时栈中只有tempGameOjbect,"Transfrom"两个元素。
4. 根据参数的数量和类型判断具体调用哪个重载的函数体
5. 通过tempGameObject代表的c#实例的索引，在objects表中找到对应的实例。同时取出"Transform"这个参数，准备进行真正的函数调用。
6. 执行obj.GetComponent(arg0)，将结果包装成一个fulluserdata后压栈，结束调用。
7. lua中的transfrom变量赋值为这个压栈的fulluserdata。
步骤3-7都在是warp文件中完成的。

### 一个类通过wrap文件注册进lua虚拟机后是什么样子的
以GameObject类举例:  
1. GameObject类：其实只是一个放在_G表中供人调用的一个充当索引的表，我们通过它来触发GameObject元表的各种元方法，实现对c#类的使用。
2. GameObject的实例：是一个fulluserdata,内容为一个整数，这个整数代表了这个实例在objects表中的索引（objects是一个用list实现的回收链表，lua中调用的c#类实例都存在这个里面），每次在lua中调用一个c#实例的方法时，都会通过这个索引找到这个索引在c#中对应的实例，然后进行操作，最后将操作结果转化为一个fulluserdata（或lua的内建类型，如bool等）压栈，结束调用。

### lua中c#实例的真正存储位置
每一个c#实例在lua中是一个内容为整数索引的fulluserdata，在进行函数调用时，通过这个整数索引查找和调用这个索引代表的实例的函数和变量。  
lua中调用和创建的c#实例实际都是存在c#中的objects表中，lua中的变量只是一个持有该c#实例索引位置的fulluserdata，并没有直接对c#实例进行引用。对c#实例进行函数的调用和变量的修改都是通过元表调用操作wrap文件中的函数进行的。  
流程图如下:



                  
