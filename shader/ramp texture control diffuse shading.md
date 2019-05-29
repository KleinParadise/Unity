### 渐变纹理控制漫反射

实现效果:  
着重强调surface的颜色，而减弱漫反射光线或其他光线的影响。用于非写实的画面,很多卡通风格的游戏中使用这种技术。不需要很多的真实物理模拟的光照模型,实现更加艺术而非写实风格的画面。  

实现该效果,需要修改[自定义漫反射光照模型](https://github.com/KleinParadise/Unity/blob/master/shader/%E8%87%AA%E5%AE%9A%E4%B9%89%E6%BC%AB%E5%8F%8D%E5%B0%84%E5%85%89%E7%85%A7%E6%A8%A1%E5%9E%8B.md)中的LightingBasicDiffuse函数，增加一个新的变量ramp：代码如下:
```HLSL
inline float4 LightingBasicDiffuse (SurfaceOutput s, fixed3 lightDir, fixed atten)
{
  float difLight = max(0, dot (s.Normal, lightDir));
  float hLambert = difLight * 0.5 + 0.5;
  float3 ramp = tex2D(_RampTex, float2(hLambert)).rgb;

  float4 col;
  col.rgb = s.Albedo * _LightColor0.rgb * (ramp);
  col.a = s.Alpha;
  return col;
}
```

_RampTex就是需要提供的渐变纹理。为了在inspector中灵活的拖拽一个texture。需要在Properties块中声明它，然后在SubShader中声明一个相同名字的变量，并制定它的类型，之后就可以在函数中访问它。完整实现代码如下:

```HLSL
Shader "Custom/RampDiffuse" {
  Properties {
    _EmissiveColor ("Emissive Color", Color) = (1,1,1,1)
    _AmbientColor  ("Ambient Color", Color) = (1,1,1,1)
    _MySliderValue ("This is a Slider", Range(0,10)) = 2.5
    // Add this line
    _RampTex ("Ramp Texture", 2D) = "white"{}
  }
  SubShader {
    Tags { "RenderType"="Opaque" }
        LOD 200

        CGPROGRAM
        #pragma surface surf BasicDiffuse

        //We need to declare the properties variable type inside of the
        //CGPROGRAM so we can access its value from the properties block.
        float4 _EmissiveColor;
        float4 _AmbientColor;
        float _MySliderValue;
        // Add this line
        sampler2D _RampTex;

        struct Input
        {
            float2 uv_MainTex;
        };

        void surf (Input IN, inout SurfaceOutput o)
        {
            //We can then use the properties values in our shader
            float4 c;
            c =  pow((_EmissiveColor + _AmbientColor), _MySliderValue);
            o.Albedo = c.rgb;
            o.Alpha = c.a;
        }

        inline float4 LightingBasicDiffuse (SurfaceOutput s, fixed3 lightDir, fixed atten)
        {
            float difLight = max(0, dot (s.Normal, lightDir));
            float hLambert = difLight * 0.5 + 0.5;
            // Add this line
            float3 ramp = tex2D(_RampTex, float2(hLambert,hLambert)).rgb;

            float4 col;
            // Modify this line
            col.rgb = s.Albedo * _LightColor0.rgb * (ramp);
            col.a = s.Alpha;
            return col;
    }

    ENDCG
  } 
  FallBack "Diffuse"
}
```

``` float3 ramp = tex2D(_RampTex, float2(hLambert,hLambert)).rgb; ```
实现该效果最关键的一行。  
这行代码返回一个rgb值。tex2D函数接受两个参数：第一个参数是操作的texture，第二个参数是需要采样的UV坐标。这里，我们并不像使用一个vertex来代表一个UV坐标进行采样，而仅仅想使用一个漫反射浮点值（即hLambert）来映射到渐变图上的某一个颜色值。最后得到的结果便是，我们将会根据计算得到的Half Lambert光照值来决定光线照射到一个物体表面的颜色变化。


