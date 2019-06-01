### 利用2d纹理模拟BRDF效果

#### BRDF(bidirectional reflectance distribution function) 双向反射分布函数  
考虑光线是如何从一个入射角度（the light direction）在一个不透明平面上反射到某一个观察者的眼睛（the view direction）里的。


```HLSL
inline float4 LightingBasicDiffuse (SurfaceOutput s, fixed3 lightDir, half3 viewDir, fixed atten)
{
    float difLight = max(0, dot (s.Normal, lightDir));
    // Add this line
    float rimLight = max(0, dot (s.Normal, viewDir));
    // Modify this line
    float dif_hLambert = difLight * 0.5 + 0.5;
    // Add this line
    float rim_hLambert = rimLight * 0.5 + 0.5;
    // Modify this line
    float3 ramp = tex2D(_RampTex, float2(dif_hLambert, rim_hLambert)).rgb;

    float4 col;
    col.rgb = s.Albedo * _LightColor0.rgb * (ramp);
    col.a = s.Alpha;
    return col;
}
```
1. 给LightingBasicDiffuse光照函数新增观察方向参数(half3 viewDir) 这个参数将会由Unity内部提供，来得到当前摄像机的观察位置到观察点的方向向量。
2. ``` float rimLight = max(0, dot (s.Normal, viewDir)); ```计算观察方向与观察物体平面的法向量夹角的余弦值
3. ``` float rim_hLambert = rimLight * 0.5 + 0.5;  ``` 使用Half Lambert方法改善rimLight的值
4. ``` float3 ramp = tex2D(_RampTex, float2(dif_hLambert, rim_hLambert)).rgb; ```通过dif_hLambert，rim_hLambert从2纹理取样。

当使用了观察方向这个参数后，我们可以创建一个非常简单的衰减渲染结果。下图显示了观察方向和平面法向量进行dot运算后的结果：  
![Image of deque](https://github.com/KleinParadise/Unity/blob/master/shader/pic/shuaijian.png)

而通过一张二维渐变图，我们可以考虑两个方向对我们观察结果的影响：  
![Image of deque](https://github.com/KleinParadise/Unity/blob/master/shader/pic/2djianbian.png)
