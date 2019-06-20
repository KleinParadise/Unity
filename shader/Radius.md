## 根据地形创建圆环

### 应用场景
游戏中选择某一物体,或者在选中的角色在其底部添加圆环,加强选中效果。游戏中的地形可能凹凸不平,圆环要根据地形显示。

### shader代码
```HLSL
Shader "CookbookShaders/Chapter02/RadiusShader" {
	Properties {
		_Color("Color", Color) = (1,1,1,1)
		_MainTex("Albedo (RGB)", 2D) = "white" {}
		
		_Center("Center", Vector) = (0,0,0,0)
		_Radius("Radius", Float) = 0.5
		_RadiusColor("Radius Color", Color) = (1,0,0,1)
		_RadiusWidth("Radius Width", Float) = 2
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
		fixed4 _Color;

		float3 _Center;
		float _Radius;
		fixed4 _RadiusColor;
		float _RadiusWidth;

		struct Input {
			float2 uv_MainTex;
			float3 worldPos;
		};

		void surf(Input IN, inout SurfaceOutputStandard o) {
			float d = distance(_Center, IN.worldPos);
			if (d > _Radius && d < _Radius + _RadiusWidth)
				o.Albedo = _RadiusColor;
			else
				o.Albedo = tex2D(_MainTex, IN.uv_MainTex).rgb;
		}
		ENDCG
	} 
	FallBack "Diffuse"
}
```
1. 在Properties中声明```_MainTex(地形纹理)，_Center(圆点),_Radius(圆环半径),_RadiusColor(圆环颜色),_RadiusWidth(圆环宽度)```等变量
2. 在subshader中添加在Properties新增的变量,进行链接
3. 修改input结构,```float2 uv_MainTex;```获得地形的纹理值，```float3 worldPos;```获得地形中每个像素在世界坐标的位置。
4. ```float d = distance(_Center, IN.worldPos);```算出当前像素点与圆环原点之间的距离。
5. ```if (d > _Radius && d < _Radius + _RadiusWidth)```如果当前像素点大于圆环的半径并且效应圆环的款，则该像素输出的颜色为圆环的颜色```_RadiusColor```
6. 如果当前像素点不在圆环的半径和圆环的宽度之间，则该像素的输出颜色为地形的颜色。```o.Albedo = tex2D(_MainTex, IN.uv_MainTex).rgb;```


### 圆环随角色移动
```c#
using UnityEngine;

[ExecuteInEditMode]
public class Radius : MonoBehaviour {

    public Material radiusMaterial;
    public float radius = 1;
    public Color color = Color.white;

    // Use this for initialization
    void Start () {
	
	}
	
	// Update is called once per frame
	void Update () {
        radiusMaterial.SetVector("_Center", transform.position);
        radiusMaterial.SetFloat("_Radius", radius);
        radiusMaterial.SetColor("_RadiusColor", color);
    }
}
```
1. 创建Radius脚本，挂在角色实体上。
2. 设置角色的圆环的半径```radius```和颜色```color```
3. 设置圆环的中心为角色的位置。```radiusMaterial.SetVector("_Center", transform.position);```
4. 设置圆环的半径和颜色。```radiusMaterial.SetFloat("_Radius", radius); radiusMaterial.SetColor("_RadiusColor", color);```












