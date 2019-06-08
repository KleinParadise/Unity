## 透明材质(纹理实现透明效果)

```HLSL
Shader "Custom/SimpleAlpha" {
	Properties {
		_MainTex ("Base (RGB)", 2D) = "white" {}
		_TransVal ("Transparency Value", Range(0,1)) = 0.5
	}
	SubShader {
		Tags { "RenderType"="Opaque" "Queue"="Transparent"}
		LOD 200
		
		CGPROGRAM
		#pragma surface surf Lambert alpha

		sampler2D _MainTex;
		float _TransVal;

		struct Input {
			float2 uv_MainTex;
		};

		void surf (Input IN, inout SurfaceOutput o) {
			half4 c = tex2D (_MainTex, IN.uv_MainTex);
			o.Albedo = c.rgb;
			o.Alpha = c.b * _TransVal;
		}
		ENDCG
	} 
	FallBack "Diffuse"
}
```
1. 在Properties中新增```_MainTex(需要实现透明的纹理)、_TransVal(控制透明度系数)```两个属性

2. ``` Tags { "RenderType"="Opaque" "Queue"="Transparent"} ``` 指定该shader的渲染队列为透明。

3. ``` #pragma surface surf Lambert alpha ``` 使用名为surf的Surface Function，使用内置的Lambert光照函数，开启**透明**通道。

4. ``` sampler2D _MainTex; float _TransVal; ```重新声明变量,以便在cg代码中使用

5. ``` o.Alpha = c.b * _TransVal; ``` 在surf()函数中添加控制透明度的代码。需要逐像素地使用一个取值范围为0到1的值来填充SurfaceOutput结构体中的
O.Alpha值。从颜色角度讲（这里的颜色指一个灰度值，因为透明度可以用一个单通道的灰度值来表示），一个为1的透明度，即白色，将会产出一个完全不透明的效果。
而0值，即黑色，表示一个完全透明的效果。

使用下图制作透明材质,这张贴图仅包含单纯的RGB和白色。使用它的RGB通道作为一个取值为0或1的透明度值。
![Image of deque](https://github.com/KleinParadise/Unity/blob/master/shader/pic/transparent.png)


``` o.Alpha = c.r * _TransVal, o.Alpha = c.g * _TransVal, o.Alpha = c.b * _TransVal）``` 对应了下图的三种效果
![Image of deque](https://github.com/KleinParadise/Unity/blob/master/shader/pic/transparent_1.png)


``` o.Alpha = c.r * _TransVal ``` 当我们使用如下语句控制透明度,则贴图中除了红色部分以及白色部分（白色的RGB通道值均为1）其R通道的值为1，
其余（绿色和蓝色部分）均为0。因此只有红色和白色的部分才不透明。
