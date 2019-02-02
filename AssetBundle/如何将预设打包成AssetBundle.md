#### 使用unity编辑器将预设打包
1. 在工程Assets文件夹中新建文件夹prefab,新建预设,如图所示  
![Image of deque](https://github.com/KleinParadise/Unity/blob/master/AssetBundle/pic/prefab.png)
2. 在工程Assets文件夹中新建文件夹script,新建build脚本。
```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEditor;

public static class Build{
	[MenuItem("Custom/Create AssetBundle")]
	static void Create(){
		//获取选中需要打包的预设
		GameObject[] gameObjList = Selection.gameObjects;
		//声明AssetBundleBuild列表
		List<AssetBundleBuild> list = new List<AssetBundleBuild> ();
		foreach (GameObject go in gameObjList) {
			//获取预设的路径
			string goPath = AssetDatabase.GetAssetPath (go);
			//设置asset 不加这段会报”Variant folder path cannot be empty.“错误
			//应该是AssetLable中没有对应的数据才会报错
			AssetImporter importer = AssetImporter.GetAtPath(goPath);
			importer.assetBundleName = go.name;
			importer.assetBundleVariant = "ab";

			//新建AssetBundleBuild
			AssetBundleBuild abBuild = new AssetBundleBuild ();
			//设置将预设打包为AssetBundle文件的名称
			abBuild.assetBundleName = go.name;
			//设置将预设打包为AssetBundle文件的后缀
			abBuild.assetBundleVariant = "ab";

			abBuild.assetNames = new string[1]{ goPath };
			list.Add (abBuild);
		};
		BuildPipeline.BuildAssetBundles (Application.dataPath + "/Resources", list.ToArray(), BuildAssetBundleOptions.None, BuildTarget.StandaloneOSXIntel64);
		AssetDatabase.Refresh ();
	}
}
```
3. 在编辑器中选择预设cube_1,cube_2,点击工具栏build脚本生成的Custom/Create AssetBundle按钮即可为选择的预设打包。每一个AssetBundle资源会有一个和文件相关的Mainfest 的文本类型的文件，该文件提供了所打包资源的CRC和资源依赖的信息。打包结果如下图
![Image of deque](https://github.com/KleinParadise/Unity/blob/master/AssetBundle/pic/assetbundle.png)

4. 在实际开发过程中,可将一个系统的资源和预设打成一个AssetBundle,将打包的资源名设置为用一个名称即可。以此来达到资源优化分类的目的。
```c#
//新建AssetBundleBuild
AssetBundleBuild abBuild = new AssetBundleBuild ();
//设置将预设打包为AssetBundle文件的名称 多个资源设置为同一名称将被打成一个AssetBundle
abBuild.assetBundleName = "设置为同一名称";
//设置将预设打包为AssetBundle文件的后缀
abBuild.assetBundleVariant = "ab";

abBuild.assetNames = new string[1]{ goPath };
list.Add (abBuild);
```
