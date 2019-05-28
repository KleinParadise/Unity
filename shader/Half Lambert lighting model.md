### 半兰伯特光照模型
实现效果:
用于提高物体在一些光线无法照射到的区域的亮度的。即通过提高了漫反射光照的亮度，使得漫反射光线可以看起来照射到一个物体的各个表面。

实现该效果,只需要修改上一章[自定义漫反射光照模型](https://github.com/KleinParadise/Unity/blob/master/shader/%E8%87%AA%E5%AE%9A%E4%B9%89%E6%BC%AB%E5%8F%8D%E5%B0%84%E5%85%89%E7%85%A7%E6%A8%A1%E5%9E%8B.md)中的LightingBasicDiffuse函数，代码如下:
```HLSL
inline float4 LightingBasicDiffuse (SurfaceOutput s, fixed3 lightDir, fixed atten)
{
    float difLight = max(0, dot (s.Normal, lightDir));
    // Add this line
    float hLambert = difLight * 0.5 + 0.5;

    float4 col;
    // Modify this line
    col.rgb = s.Albedo * _LightColor0.rgb * (hLambert * atten * 2);
    col.a = s.Alpha;
    return col;
}
```
通过定义了一个新的变量hLambert来替换difLight用于计算某点的颜色值。difLight的范围是0.0-1.0，而通过hLambert，我们将结果由0.0-1.0映射到了0.5-1.0，
从而达到了增加亮度的目的。

下图是difLight与hLambert数值曲线  
![Image of deque](https://github.com/KleinParadise/Unity/blob/master/shader/pic/Half%20Lambert%20dot.png)


Lambert与Half Lambert效果对比  
![Image of deque](https://github.com/KleinParadise/Unity/blob/master/shader/pic/Half%20Lambert.png)

