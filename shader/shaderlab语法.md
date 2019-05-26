### 通过创建的默认shader分析其语法

```HLSL
Shader "Custom/Diffuse Texture" {
	Properties {
		_MainTex ("Base (RGB)", 2D) = "white" {}
	}
	SubShader {
		Tags { "RenderType"="Opaque" }
		LOD 200
		
		CGPROGRAM
		#pragma surface surf Lambert

		sampler2D _MainTex;

		struct Input {
			float2 uv_MainTex;
		};

		void surf (Input IN, inout SurfaceOutput o) {
			half4 c = tex2D (_MainTex, IN.uv_MainTex);
			o.Albedo = c.rgb;
			o.Alpha = c.a;
		}
		ENDCG
	} 
	FallBack "Diffuse"
}
```

#### 属性
每一条属性都会作为输入提供给所有的子着色器。  
声明属性的语法如下:  
```HLSL
_Name("Display Name", type) = defaultValue[{options}]
```
``` _Name ```该属性申请的变量名。整个shader用这个变量名获取该属性。  
``` "Display Name" ```该字符串将显示在Unity的Inspector中作为Shader的使用者可读的内容,即显示的名称。  
``` type ``` 属性的类型。类型有如下几种：  

type | desc
------------ | -------------
Color | 一种颜色，由RGBA（红绿蓝和透明度）四个量来定义
2D | 一张2的阶数大小（256，512之类）的贴图。这张贴图将在采样后被转为对应基于模型UV的每个像素的颜色，最终被显示出来
Rect | 一个非2阶数大小的贴图；
Cube | 即Cube map texture（立方体纹理），简单说就是6张有联系的2D贴图的组合，主要用来做反射效果（比如天空盒和动态反射），也会被转换为对应点的采样
Range(min, max) | 一个介于最小值和最大值之间的浮点数，一般用来当作调整Shader某些特性的参数（比如透明度渲染的截止值可以是从0至1的值等）
Float | 任意一个浮点数
Vector | 一个四维数

```defaultValue ``` 表示当前属性类型的默认值  

type | defaultValue
------------ | -------------
Color | 以0～1定义的rgba颜色，比如(1,1,1,1)
2D、Rect、Cube | 默认值可以为一个代表默认tint颜色的字符串，可以是空字符串或者是某个内置的缺省纹理”white”,”black”,”gray”,”bump”中的一个
Range(min, max)、Float | 某个指定的浮点数
Vector | 一个四维数

```{options} ``` 只对2D，Rect或者Cube贴图有关，在写输入时我们最少要在贴图之后写一对什么都不含的空白的{}，当我们需要打开特定选项时可以把其写在这对花括号内。如果需要同时打开多个选项，可以使用空白分隔。可能的选择有ObjectLinear, EyeLinear, SphereMap, CubeReflect, CubeNormal中的一个。

#### Tags
硬件将通过判定这些标签来决定什么时候调用该着色器。  

Tags | desc
------------ | -------------
Tags { "RenderType"="Opaque" } | 在渲染非透明物体调用该着色器
Tags { "RenderType"="Transparent" } | 渲染含有透明效果的物体时调用该着色器
Tags { "IgnoreProjector"="True" } | 不被Projectors影响
Tags { "ForceNoShadowCasting"="True" } | 从不产生阴影
Tags { "Queue"="xxx" } | 指定渲染顺序队列

#### LOD
是Level of Detail的缩写,这个数值决定了我们能用什么样的Shader。在Unity的Quality Settings中我们可以设定允许的最大LOD，当设定的LOD小于SubShader所指定的LOD时，这个SubShader将不可用。  
在这里例子里我们指定了其为200（其实这是Unity的内建Diffuse着色器的设定值）。


#### Shader本体

``` CGPROGRAM ```  是一个开始标记，表明从这里开始是一段CG程序。与最后一行的ENDCG与CGPROGRAM是对应的，表明CG程序到此结束。  

``` #pragma surface surf Lambert ``` 编译指令。表示声明了一个表面着色器，实际的代码在surf函数中（在下面能找到该函数），使用Lambert（也就是普通的diffuse）作为光照模型  
- surface - 声明的是一个表面着色器
- surfaceFunction - 着色器代码的方法的名字
- lightModel - 使用的光照模型。

``` sampler2D _MainTex; ``` 
加载以后的texture（贴图）不过是一块内存存储的，使用了RGB（也许还有A）通道，且每个通道8bits的数据。而具体地想知道像素与坐标的对应关系，以及获取这些数据，我们总不能一次一次去自己计算内存地址或者偏移，因此可以通过sampler2D来对贴图进行操作。更简单地理解，sampler2D就是GLSL中的2D贴图的类型，相应的，还有sampler1D，sampler3D，samplerCube等等格式。  
实例的这个shader其实是由两个相对独立的块组成的，外层的属性声明，回滚等等是Unity可以直接使用和编译的ShaderLab；而现在我们是在``` CGPROGRAM...ENDCG ```这样一个代码块中，这是一段CG程序。对于这段CG程序，要想访问在Properties中所定义的变量的话，必须使用和之前变量相同的名字进行声明。于是其实sampler2D _MainTex;做的事情就是再次声明并链接了_MainTex，使得接下来的CG程序能够使用这个变量。  

上面的#pragma段已经指出了着色器代码的方法的名字叫做surf，因此surf函数就是该着色器的工作核心。着色器就是给定了输入，然后给出输出进行着色的代码。CG规定了声明为表面着色器的方法（就是我们这里的surf）的参数类型和名字，因此我们没有权利决定surf的输入输出参数的类型，只能按照规定写。这个规定就是第一个参数是一个Input结构，第二个参数是一个inout的SurfaceOutput结构。  
Input其实是需要我们去定义的结构，这给我们提供了一个机会，可以把所需要参与计算的数据都放到这个Input结构中，传入surf函数使用。  
```HLSL
//Input 结构体
struct Input {
	float2 uv_MainTex;
};
```
SurfaceOutput是已经定义好了里面类型输出结构，但是一开始的时候内容暂时是空白的，我们需要向里面填写输出，这样就可以完成着色了。  
```HLSL
//SurfaceOutput 结构体
struct SurfaceOutput {
    half3 Albedo;     //像素的颜色
    half3 Normal;     //像素的法向值
    half3 Emission;   //像素的发散颜色
    half Specular;    //像素的镜面高光
    half Gloss;       //像素的发光强度
    half Alpha;       //像素的透明度
};
```

```HLSL
void surf (Input IN, inout SurfaceOutput o) {
  half4 c = tex2D (_MainTex, IN.uv_MainTex);
  o.Albedo = c.rgb;
  o.Alpha = c.a;
}
```

在CG程序中，我们有这样的约定，在一个贴图变量（即例子中是_MainTex）之前加上uv两个字母，就代表提取它的uv值（其实就是两个代表贴图上点的二维坐标 ）。我们之后就可以在surf程序中直接通过访问uv_MainTex来取得这张贴图当前需要计算的点的坐标值了。  
在计算输出时Shader会多次调用surf函数，每次给入一个贴图上的点坐标(赋值uv_MainTex)，来计算输出。第二个参数是一个可写的SurfaceOutput，SurfaceOutput是预定义的输出结构，我们的surf函数的目标就是根据输入把这个输出结构填上。  
使用tex2d函数，它是CG程序中用来在一张贴图中对一个点进行采样的方法，返回一个float4。这里对_MainTex在输入点上进行了采样，并将其颜色的rbg值赋予了输出的像素颜色，将a值赋予透明度。即该着色器工作的工作：找到贴图上对应的uv点，直接使用颜色信息来进行着色。


