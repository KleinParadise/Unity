### Assets目录下文件夹的作用
##### Resources
Resources文件夹可以放在Assets的根目录下,也可放在子目录中。/xxx/xxx/Resources  和 /Resources 是一样的，无论多少个叫Resources的文件夹都可以。
所有Assets目录下的Resources文件夹中的内容都会被打进发布版本中。  
可以使用Resources.Load()可在编辑器与真机上使用。  
Resources.LoadAssetAtPath()、AssetDatabase.LoadAssetAtPath()这两个函数可以读取Assets目录下的任意文件夹下的资源，但是只能在编辑时用。
##### Editor
存放编辑器的扩展脚本。Editor文件件可以在根目录，也可以在子目录里。Editor下放的资源不会被打进发布包中，脚本只能在编辑时使用。
##### Plugins
手机游戏开发一般将要接的sdk依赖的库文件放入对应文件夹中，例如Plugins/Android、Plugins/iOS
##### StreamingAssets
该文件夹目录下的所有资源都会打入发布包中。他与Resources的区别是，Resources会压缩文件，但StreamingAssets中的文件不会被压缩。StreamingAssets是一个只读的文件夹，程序运行时只能读不能写，各平台的路径可以使用Application.streamingAssetsPath获取。初始的assetbundle放在StreamingAssets目录下，运行程序时，将这些assetbundle拷贝到Application.persistentDataPath目录下，如果有更新的话，直接覆盖Application.persistentDataPath目录下原有的资源。
Application.persistentDataPath目录是应用程序的沙盒目录，打包之前不存在该目录，直到应用程序安装完成。
##### Scripts
存放游戏脚本
##### Scenes
存放游戏场景


### Resources与StreamingAssets的区别
Resources | StreamingAssets
------------ | -------------
只读文件夹 | 只读文件夹
文件夹中的内容都会被打进发布版本中 | 文件夹中的内容都会被打进发布版本中
Resources.Load()加载 | 通过www方式加载
压缩打包 | 原封不动的打包
加密 | 不会被加密
