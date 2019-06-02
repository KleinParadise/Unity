```HLSL
Shader "Custom/ScrollingUVs" {
	Properties {
		_MainTex ("Base (RGB)", 2D) = "white" {}
		// Add two properties
		_ScrollXSpeed ("X Scroll Speed", Range(0, 10)) = 2
		_ScrollYSpeed ("Y Scroll Speed", Range(0, 10)) = 2
	}
	SubShader {
		Tags { "RenderType"="Opaque" }
		LOD 200
		
		CGPROGRAM
		#pragma surface surf Lambert

		sampler2D _MainTex;
		fixed _ScrollXSpeed;
		fixed _ScrollYSpeed;

		struct Input {
			float2 uv_MainTex;
		};

		void surf (Input IN, inout SurfaceOutput o) {
			fixed2 scrolledUV = IN.uv_MainTex;
			
			fixed xScrollValue = _ScrollXSpeed * _Time.y;
			fixed yScrollValue = _ScrollYSpeed * _Time.y;
			
			scrolledUV += fixed2(xScrollValue, yScrollValue);
			
			half4 c = tex2D (_MainTex, scrolledUV);
			o.Albedo = c.rgb;
			o.Alpha = c.a;
		}
		ENDCG
	} 
	FallBack "Diffuse"
}
```

1. 新增 ```_ScrollXSpeed, _ScrollYSpeed  ```两个Properties, 使我们可通过这两个变量调整texture的滚动速度。  

2. 在CGPROGRAM部分修改代码，添加两个新的变量，``` fixed _ScrollXSpeed; fixed _ScrollYSpeed; ```
对应上面新增的两个Properties，以使我们可以在后面cg代码访问它们：

3. 在surf函数中，首先将UV坐标存储在scrolledUV变量中，并且该变量需要是float2类型或者fixed2类型。这是因为是通过以下定义的结构来传递UV的：
``` struct Input { float2 uv_MainTex;};```随后，通过内置变量_Time计算UV偏移量。_Time变量返回一个float4类型的变量。最后，将计算而得的偏移量叠加到之前得到的UV坐标scrolledUV上，得到最终的UV坐标，并通过tex2D函数访问该像素值。
