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


                  
