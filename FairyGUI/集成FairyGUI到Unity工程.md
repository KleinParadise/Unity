### FairyGUI集成到unity工程
1. 从FairyGUI官网下载FairyGUI-unity-master.zip包并解压。
2. 在当前unity工程的Assets目录下新建FairyGUI文件夹。
3. 将FairyGUI-unity-master/Assets/Examples/Resources文件夹拷贝至FairyGUI文件夹中。
4. 将FairyGUI-unity-master/Assets/Examples/Scripts文件夹拷贝至FairyGUI文件夹中。
5. 在当前unity工程场景中右键新增FairyGUI/UIPanel组件并保存场景。
6. 选中UIPanel组件,Inspector中可设置UIPanel组件要运行的包名(Package Name)和组件名(Component Name)即加载FairyGUI编辑器创建的文件。包名(Package Name)和组件名(Component Name)读取的是FairyGUI/Resources下的文件。

### unity工程中如何使用FairyGUI
1. 将FairyGUI-unity-master/Assets/Examples/Bag文件夹拷贝到当前工程FairyGUI/Scripts文件夹中。
2. 选中UIPanel组件,在该组件上挂载FairyGUI/Scripts/Bag/BagMain.cs脚本组件
3. 运行游戏场景,即可看到FairyGUI自带的Examples/Bag示例可在当前工程运行


