### 概述

- **Material与UnityShader**
  - 两者之间的关系
    - UnityShader定义了渲染所需的各种代码(如顶点&片段着色器),属性(如使用哪些纹理)和指令(渲染和标签设置等),材质则允许我们调节这些属性,并将其赋值给相应的模型.
  - 材质
    - 材质需要结合一个GameObjec的Mesh或Particle Systems组件来工作,它与shader配合决定了游戏对象看起来的样子.
    - Unity5.x版本中新建一个材质会默认使用内置的基于物理渲染的着色器Standard Shader.
  - Mesh
    - 在Unity中Mesh用来描述物体的形状.
    - 主要属性内容包括顶点坐标，法线，纹理坐标，三角形绘制序列等其他有用属性和功能.
  - UnityShader
    - Unity提供的4种shader模板
      - Standard Sruface Shader 包含了标准光照模型(基于物理的渲染方法)表面着色器模板
      - Unlit Shader 不包含光照(包括雾效)的基本顶点/片段着色器
      - Image Effect Shader 为实现各种屏幕后处理效果提供一个基本模板
      - Compute Shader会产生一种特殊的shader文件旨在利用GPU的并行进行与常规流水线无关的运算
  - Unity编辑器中查看shader
      - shader的属性面板中可查看shader的相关信息,如是否是一个表面着色器,是否是一个固定函数着色器,还可以查看一些标签设置如是否会投射阴影,使用的渲染队列,LOD值等



- **ShaderLab**
  - what shaderLab
    - 定义了要显示一个材质所需的东西,不仅包含着色器代码,也包含许多渲染所需的数据

  - Unity how to use shaderLab
    - Unity在背后会根据使用的平台把shaderLab结构编译成代码和shader文件 



- **UnityShader结构**
  - name
    - UnityShader文件的第一行都需要通过Shader语义指定改UnityShader的名字,这些名称会出现在材质面板的下拉列表里,通过"/"来控制目录结构
    ```cg
      Shader "Custom/MyShader"
    ```
    
  - Properties
    - 材质与UnityShader直接的桥梁
    ```cg
      Propertise{
        Name("display name",PropertyType) = DefaultValue
      }
      //Name 在shader中访问该属性通过Name,属性名字通常由一个下划线开始
      //"display name" 出现在材质面板的显示名称
      //PropertyType 为该属性指定的类型
      //DefaultValue 为该属性指定一个默认值
    ```
    - Properties语义支持的类型
      属性类型 | 默认值的定义语法 | 例子
      ------------ | -------------  | -------------
      Int | number | _Int("Int",Int) = 2
      Float | number | _Float("Float",Float) = 1.5
      Range(min,max) | number | _Range("Range",Range(0.0,5.0)) = 3.0
      Color | (number,number,number,number) | _Color("Color",Color) = (1,1,1,1)
      Vector | (number,number,number,number) | _Vector("Vector",Vector) = (2,3,6,1)
      2D | "defaulttexture" {} | _2D("2D",2D) = "" {}
      Cube | "defaulttexture" {} | _Cube("Cube",Cube) = "white" {}
      3D | "defaulttexture" {} | _3D("3D",3D) = "black" {}
      
    - 2D,Cube,3D纹理类型
      - 通过一个空字符串和一个花括号指定,字符串要不是空的要么是内置的纹理名称("white","black","gray","bump")花括号用于指定一些纹理属性5.0以后的版本被移除

    - shader中如何访问这些属性
      - 需要在CG代码片中定义和这些属性类型相匹配的变量
      - 不在Propertiss中声明这些属性,也可以在CG代码片定义变量,此时通过脚本向Shader中传递这些属性
      - Properties语义块的作用仅为了这些属性出现在材质面板中


  - SubShader
    - 为什么一个UnityShader里面可存在多个SubShader
      - 支持多个显卡,旧显卡使用计算复杂度较低的SubShader,高级显卡使用计算复杂度较高的SubShader
      - Unity会扫描所有的SubShader语义块,选择第一个能够在目标平台上运行的SubShader

    - SubShader语义块
      ```CG
        SubShader{
          [Tags]
          [RenderSetup]
          Pass{
          }
          //ohter Passes
        }
      ```
      
    - Tag
      - 键值对,键和值都是字符串类型
      - 通过Tag键值对在SubShader与渲染引擎中架起桥梁,即通过Tag告诉渲染引擎SubShader希望怎样以及何时渲染这个对象
      - Tag结构
        ```cg
          Tags{"TarName1" ="Value1" "TarName2" ="Value2"}
        ```        
      - [更多SubShader Tags介绍](https://github.com/KleinParadise/Unity/blob/master/shader/%E5%85%A5%E9%97%A8%E7%B2%BE%E8%A6%81/SubShader%20Tags.md)
        
    - RenderSetup
      - 设置显卡的各种状态,并将应用到所有的pass上,如果只想在某一个Pass设置该状态,可在Pass语义块中进行单独设置
      - 渲染状态设置选项 
        状态名称 | 设置指令 | 解释
        ------------ | -------------  | -------------
        Cull | CullBack/Front/Off  | 设置剔除模式，剔除背面/正面/关闭剔除
        ZTest | ZTestLess Greater/LEqual/GEqual/NotEqual/Always | 设置深度测试时使用的函数
        ZWrite | ZWrite On/Off | 开启/关闭深度写入
        Blend | Blend SrcFactor DstFactor | 开启并设置混合模式
        
        
        
    - Pass
      - 语义
        ```CG
          Pass {
            [Name]
            [Tags]
            [RenderSetup]
            //other code
          }
          //
          //
        ```
      - Name  
        - Name 定义该Pass的名称,通过该名称可以使用UsePass命令使用其他UnityShader的Pass,使用Pass是要使用大写形式的Name
      - RenderSetup
        - SubShader的RenderSetup同样适用与Pass
      - Tags
        - Pass的Tag不用于SubShader的Tags,作用也是告诉渲染引擎怎么样渲染该物体 
        - Pass标签的类型
          标签类型 | 说明 | 例子
          ------------ | -------------  | -------------
          LightMode | 定义该Pass在Unity的渲染流水线的角色  | Tags{"LightMode" = "ForwardBase"}
          RequireOptions | 用于指定满足某些条件才渲染该Pass | Tags{"RequireOptions" = "SoftVegetation"}
          
          
    - FallBack
      - 用于告诉Unity,如果上面所有的SubShader在这块显卡上都不能运行,那就使用这个默认shader
      - 语义
        ```CG
          FallBack "name"
          //也可以关闭FallBack功能
          FallBack off
        ```

- **UnityShader的形式**
  - 表面着色器
    - Unity自己创造的一种着色器类型,使用该着色器Unity会默认处理很多光照细节
    - 定义在SubShader语义块中的CGPROGRAM和ENDCG之间,而并不是定义在Pass块的CGPROGRAM和ENDCG之间
    - Unity会将其转化为一个包含多个Pass的顶点/片段着色器
  - 顶点/片段着色器
    - 定义在Pass块的CGPROGRAM和ENDCG之间
    - 灵活性最高,可以用来控制渲染的细节,需要处理的东西更多
  - 固定函数着色器
    - 定义在Pass语义块中,只能使用shaderLab的渲染设置命令来编写,不能使用CG/HLSL
  - 如何选择?  
    - 与光源有关,使用表面着色器,要关注性能
    - 光照少,实现自定义的渲染效果,使用顶点/片段着色器
    - 老设备固定管线着色器
