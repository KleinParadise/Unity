### SubShader Tags支持的标签类型
标签类型 | 说明 | 例子
------------ | -------------  | -------------
QUEUE | 控制渲染顺序,指定该物体属于哪一个渲染队列,通过这种方式可以保证所有的透明物体在所有的不透明物体后面渲染 | Tags{"Queue" = "Transparent"}
RenderType | 对一个着色器进行分类,如指定该着色器是透明着色器还是不透明着色器 | Tags{"RenderType" = "Opaque"}
DisableBatching | 通过该标签指明是否对该SubShader使用批处理(如在模型空间的坐标进行顶点动画) | Tags{"DisableBatching" = "True"}
ForceNoShadowCasting | 如果标签值为True那么使用该SubShader的物体不会受Projector的影响,通常用于半透明物体 | Tags{"ForceNoShadowCasting" = "True"}
IgnoreProjector | 如设置为Ture,使用该SubShader的物体不会受Projector组件的影响 | Tags{"IgnoreProjector" = "True"}
CanUseSpriteAtlas | 当该SubShader用于精灵时,设置为False | Tags{"CanUseSpriteAtlas" = "False"}
PreviewType | 设置材质面板如何预览该材质,默认为球体,对shader本身没影响 | Tags{"PreviewType" = "Plane"}



#### QUEUE
取值 | 说明 | 例子
------------ | -------------  | -------------
Background | 值为1000，此队列的对象最先进行渲染 | Tags{"Queue" = "Background"}
Geometry | 值为2000，通常用于不透明对象，比如场景中的物件与角色等 | Tags{"Queue" = "Geometry"}
AlphaTest | 值为2450，要么完全透明要么完全不透明，多用于利用贴图来实现边缘透明的效果 | Tags{"Queue" = "AlphaTest"}
Transparent | 值为3000，常用于半透明对象，渲染时从后往前进行渲染，建议需要混合的对象放入此队列 | Tags{"Queue" = "Transparent"}
Overlay | 值为4000,此渲染队列用于叠加效果。最后渲染的东西应该放在这里（例如镜头光晕) | Tags{"Queue" = "Overlay"}
通过在值后加数字的形式来实现自定义渲染队列
```CG
Tags{"Queue" = "Geometry"}
```
避免重复渲染,谨慎旋转渲染队列  
  - 渲染队列直接影响性能中的重复绘制，合理的队列可极大的提升渲染效率。在Unity中，渲染队列小于2500的对象都被认为是不透明的物体,（如“Background”，“Geometry”，“AlphaTest”），这些物体是从前往后绘制的，而使用其他的队列（如“Transparent”，“Overlay”）的物体则是从后往前绘制的。这意味着，我们需要尽可能地把物体的队列设置为不透明物体的渲染队列，而尽量避免重复绘制。


#### RenderType
利用Camera.SetReplacementShader来更改最终的渲染效果
```c#
//EffectShader是我们需要渲染的Shader，Tag是用来筛选渲染对象用的。
camera.SetReplacementShader (EffectShader, Tag);
```
使用规则:  
1. 使用SetReplacementShader（"shaderA",""）,这里Tag为空，表示全部物体Shader都替换成shaderA进行渲染。
2. SetReplacementShader("shaderA","RenderType")，由于这里Tag为"RenderType",所以先查看场景中的shader中是否有RenderType参数，如果有，再看RenderType中的值是否与shaderA中的RenderType值相等，相等则使用shaderA渲染，否则就不渲染（Game视图什么也没有）。
3. 关于Tag参数并不局限于RenderType,我们同样可以使用SetReplacementShader("shaderA","IgnoreProjector")或是其他任何参数如：SetReplacementShader("shaderA","ABC")等。

RenderType使用示例  
将场景中的某一个物体shader改为指定的shader  
```c#
camera.SetReplacementShader (ShaderFind("xxx/overdraw"), "RenderType");
```
overdraw Shader Tags定义
```CG
SubShader{
  Tags{
    "RenderType" = "needOverDraw"
  }
}
```
需要替换shader物体原本的Shader
```CG
SubShader{
  Tags{
    "Queue" = "Geometry"
    "RenderType" = "needOverDraw"
    "ForceNoShadowCasting" = "True"
  }
}
```
在场景中查找所有的对象，筛选出有RenderType标签的对象，然后进一步和xxx/Overdraw这个Shader进行对比，现需要替换的物体的值与此相同(needOverDraw)，最后使用overdraw来渲染该物体。



#### DisableBatching
  - 批处理会将所有几何转换到世界坐标空间,如果想利用模型的本地空间做一些位移变换效果,就需要关闭批处理
  - LODFading，仅当LOD激活时禁用批处理


#### ForceNoShadowCasting
  - 是否强制关闭投射阴影

#### IgnoreProjector
  - 是否忽略Projector投影器的影响，Projector是Unity中内置的组件,可用来实现模糊阴影或者弹孔等类似效果

#### CanUseSpriteAtlas
  - 如果某个图片被设置进某个图集中,而此图片的SubShader Tags被设置为"False",会导致改shader无法正常工作

#### PreviewType
  - 设置材质面板的预览窗口如何显示模型，默认显示的是球体,对Shader本身没有什么影响。
  - Plane 平面预览
  - Skybox 天空盒预览

