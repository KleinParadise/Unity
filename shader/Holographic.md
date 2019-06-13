## 全息着色器

### 正交
向量a和向量b垂直,此时ab正交。

### 透明度
0为透明 1为不透明

```HLSL
Shader "CookbookShaders/Chapter02/Silhouette" {
	Properties{
		_Color("Color", Color) = (1,1,1,1)
		_MainTex("Albedo (RGB)", 2D) = "white" {}

		_DotProduct("Rim effect", Range(-1,1)) = 0.25
	}
	SubShader{
		Tags{
			"Queue" = "Transparent"
			"IgnoreProjector" = "True"
			"RenderType" = "Transparent"
		}
		LOD 200

		Cull Off

		CGPROGRAM
		#pragma surface surf Lambert alpha:fade nolighting

		sampler2D _MainTex;
		fixed4 _Color;

		float _DotProduct;

		struct Input {
			float2 uv_MainTex;
			float3 worldNormal;
			float3 viewDir;
		};


		//void surf(Input IN, inout SurfaceOutputStandard o) {
		void surf(Input IN, inout SurfaceOutput o) {
			float4 c = tex2D(_MainTex, IN.uv_MainTex) * _Color;
			o.Albedo = c.rgb;

			float border = 1 - (abs(dot(IN.viewDir, IN.worldNormal)));
			float alpha = (border * (1 - _DotProduct) + _DotProduct);
			o.Alpha = c.a * alpha;
		}
		ENDCG
	}
	FallBack "Diffuse"
}
```

1. ``` _DotProduct("Rim effect", Range(-1,1)) = 0.25 ```Properties新增_DotProduct属性，用来判断点积需要多么接近于0时才将三角形视为其轮廓。

2. CGPROGRAM中添加```float _DotProduct;```添加对应变量

3. 因为材质透明 添加```Tags{ "Queue" = "Transparent" "IgnoreProjector" = "True" "RenderType" = "Transparent"}```该标签

4. ```#pragma surface surf Lambert alpha:fade nolighting``` 指定为Lambert光照模型，alpha:fade告知cg这是一个透明着色器，通过nolighting来禁用所
有光照。

5. ```	struct Input { float2 uv_MainTex; float3 worldNormal;float3 viewDir;}; ```定义Input结构，让unity在帮我们填充worldNormal(世界法线)和
viewDir(观察方向)

6. ``` float4 c = tex2D(_MainTex, IN.uv_MainTex) * _Color;```获取纹理的颜色,并将rgb值赋值给输出颜色o.```o.Albedo = c.rgb;```

7. ``` dot(IN.viewDir, IN.worldNormal) ``` 判断两法线是否正交。正交为1

8. ``` abs(dot(IN.viewDir, IN.worldNormal)) ```前方正交,背面取绝对值同前方正交。目的是让观察方向正面与背面都透明

9. ``` float border = 1 - (abs(dot(IN.viewDir, IN.worldNormal))); ```正交为0透明，非正交不透明,显示出轮廓

10. ``` float alpha = (border * (1 - _DotProduct) + _DotProduct);``` 根据_DotProduct值计算出最终透明度

11. ``` o.Alpha = c.a * alpha;```将纹理的透明度与计算出的透明度混合赋值给输出o的透明度
