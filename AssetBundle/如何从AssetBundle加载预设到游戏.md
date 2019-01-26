1. 在工程Assets文件夹中新建文件夹script,新建c#脚本LoadAbScript。代码如下
```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class LoadAbScript : MonoBehaviour {
	//pc平台资源加载方式
	public const string FILE_SIGN = "file://";

	void OnGUI(){
		//新建LoadGo按钮,点击执行assetBundle实例操作
		if(GUILayout.Button("LoadGo")){
			Debug.Log (" button click enter ");
			//assetBundle资源所在路径
			string path = FILE_SIGN + Application.dataPath + "/Resources/";
			//开启协同,开始加载资源
			StartCoroutine (LoadGo (path));
		}
	}
		
	private IEnumerator LoadGo(string path){
		WWW wwwObj = new WWW (path + "cube1.ab");
		yield return wwwObj;
		//加载资源
		Object obj = wwwObj.assetBundle.LoadAsset ("Cube1");
		//实例化资源
		yield return Instantiate (obj);
	}
}
```
2. 点击LoadGo按钮,实例化Cube1成功。


