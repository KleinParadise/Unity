#### 1.AssetBundle的作用？
把多个自定义的游戏对象或者资源以二进制形式保存到Assetbundle文件中。利用此功能我们可以在游戏中将一系列关联的内容制作成一个prefab,如将一个模型和其贴图和
动作制作为一个预设,然后将该预设导出为AssetBundle文件。Unity会自己收集该Prefab使用到的关联文件，将其一并打入AssetBundle文件，并保留prefab中资源和脚本
之间相互关联。从而达到对游戏资源进行的管理和对游戏进行资源更新的目的。


