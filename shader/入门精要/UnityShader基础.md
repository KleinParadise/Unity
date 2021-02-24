### 概述

- **Material与UnityShader**
  - 两者之间的关系
    - UnityShader定义了渲染所需的各种代码(如顶点&片段着色器),属性(如使用哪些纹理)和指令(渲染和标签设置等),材质则允许我们调节这些属性,并将其赋值给相应的模型.
  - 材质
    - 材质需要结合一个GameObjec的Mesh或Particle Systems组件来工作,它与shader配合决定了游戏对象看起来的样子.
    - Unity5.x版本中新建一个材质会默认使用内置的基于物理渲染的着色器Standard Shader.
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
