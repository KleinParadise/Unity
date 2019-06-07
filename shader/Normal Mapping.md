## 法线贴图

### 什么是法线贴图？产生的原因？
一个场景中有多个物体,这些物体由多个多边形组成,而这些多边形又由多个三角形组成。为了提升真实感,隐藏多边形细节,一般以向三角形上附加纹理的方式来增加额外细节,
提升效果。但是现实物体都是凹凸不平的,当我们靠近看这些使用纹理提升细节的物体时,效果则严重失真。  

如果用很密集的三角形去表示这类略有凹凸的表面，则性能上大大下降。研究人员发现，人眼对物体的凹凸感觉，很大程度上取决于物体表面的光照明暗变化。而光照与物体的
明暗变化则与垂直于该物体的法线向量有关。  

每个fragment使用了自己的法线，我们就可以让光照相信一个表面由很多微小的（垂直于法线向量的）平面所组成，物体表面的细节将会得到极大提升。这种每个fragment使
用各自的法线，替代一个面上所有fragment使用同一个法线的技术叫做法线贴图（normal mapping）或凹凸贴图（bump mapping）  



### 向量(x,y,z)到纹理(rgb)？
一条法线是一个三维向量，一个三维向量由x, y, z等3个分量组成，通过把(x, y, z)当作RGB3个颜色的值即可把法线数据存储在一个2D纹理中。  
法线向量的范围在-1到1之间，通过 ``` vec3 rgb_normal = normal * 0.5 - 0.5; ```可将其映射到0到1的范围。再转换的rgb值,从而产生纹理贴图。



### 切线空间


### unity shader 中使用法线贴图

```HLSL
Shader "CookbookShaders/Chapter02/NormalShader" {
	Properties {
		_Color ("Color", Color) = (1,1,1,1)
		_MainTex ("Albedo (RGB)", 2D) = "white" {}

		_BumpTex("Normal map", 2D) = "bump" {}
		_BumpIntensity("Normal intensity", Range(0,1)) = 1

		_Glossiness ("Smoothness", Range(0,1)) = 0.5
		_Metallic ("Metallic", Range(0,1)) = 0.0
	}
	SubShader {
		Tags { "RenderType"="Opaque" }
		LOD 200
		
		CGPROGRAM
		// Physically based Standard lighting model, and enable shadows on all light types
		#pragma surface surf Standard fullforwardshadows

		// Use shader model 3.0 target, to get nicer looking lighting
		#pragma target 3.0

		sampler2D _MainTex;
		
		sampler2D _BumpTex;
		fixed _BumpIntensity;

		struct Input {
			float2 uv_MainTex;
		};

		half _Glossiness;
		half _Metallic;
		fixed4 _Color;

		void surf (Input IN, inout SurfaceOutputStandard o) {
			// Albedo comes from a texture tinted by color
			fixed4 c = tex2D (_MainTex, IN.uv_MainTex) * _Color;
			o.Albedo = c.rgb;

			fixed4 n = tex2D(_BumpTex, IN.uv_MainTex) * _BumpIntensity;
			o.Normal = UnpackNormal(n).rgb;

			// Metallic and smoothness come from slider variables
			o.Metallic = _Metallic;
			o.Smoothness = _Glossiness;
			o.Alpha = c.a;
		}
		ENDCG
	} 
	FallBack "Diffuse"
}
```
1. 在Properties属性新增```_MainTex(主纹理)、_BumpTex(纹理贴图)、_BumpIntensity(纹理映射光强)```属性

2. 将这些属性链接到cg程序。即在CGPROGRAM后声明这些属性。```sampler2D _MainTex; sampler2D _BumpTex; fixed _BumpIntensity;```

3. ``` fixed4 n = tex2D(_BumpTex, IN.uv_MainTex) * _BumpIntensity; ```用tex2D函数从纹理贴图中获取法线纹理数据,并乘以对应的纹理映射光强值。

4. ``` o.Normal = UnpackNormal(n).rgb; ``` 通过UnpackNormal函数从法线纹理数据中提取法线信息,并赋值给输出o的法线。

5. 光照与物体的各顶点法线向量模拟出真实的凹凸效果

