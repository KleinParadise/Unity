### 源码分析
LuaFramework框架的热更写在GameManager.cs文件中。
```c#
public void CheckExtractResource() {
    bool isExists = Directory.Exists(Util.DataPath) &&
      Directory.Exists(Util.DataPath + "lua/") && File.Exists(Util.DataPath + "files.txt");
    if (isExists || AppConst.DebugMode) {
        StartCoroutine(OnUpdateResource());
        return;   //文件已经解压过了，自己可添加检查文件列表逻辑
    }
    StartCoroutine(OnExtractResource());    //启动释放协成 
}

/// <summary>
/// 启动更新下载，这里只是个思路演示，此处可启动线程下载更新
/// </summary>
IEnumerator OnUpdateResource() {
    if (!AppConst.UpdateMode) {
        OnResourceInited();
        yield break;
    }
    string dataPath = Util.DataPath;  //数据目录
    string url = AppConst.WebUrl;//资源http下载地址
    string message = string.Empty;
    string random = DateTime.Now.ToString("yyyymmddhhmmss");
    string listUrl = url + "files.txt?v=" + random;
    Debug.LogWarning("LoadUpdate---->>>" + listUrl);

    WWW www = new WWW(listUrl); yield return www;
    if (www.error != null) {
        OnUpdateFailed(string.Empty);
        yield break;
    }
    if (!Directory.Exists(dataPath)) {
        Directory.CreateDirectory(dataPath);
    }
    File.WriteAllBytes(dataPath + "files.txt", www.bytes);
    string filesText = www.text;
    string[] files = filesText.Split('\n');

    for (int i = 0; i < files.Length; i++) {
        if (string.IsNullOrEmpty(files[i])) continue;
        string[] keyValue = files[i].Split('|');
        string f = keyValue[0];
        string localfile = (dataPath + f).Trim();
        string path = Path.GetDirectoryName(localfile);
        if (!Directory.Exists(path)) {
            Directory.CreateDirectory(path);
        }
        string fileUrl = url + f + "?v=" + random;
        bool canUpdate = !File.Exists(localfile);
        if (!canUpdate) {
            string remoteMd5 = keyValue[1].Trim();
            string localMd5 = Util.md5file(localfile);
            canUpdate = !remoteMd5.Equals(localMd5);
            if (canUpdate) File.Delete(localfile);
        }
        if (canUpdate) {   //本地缺少文件
            Debug.Log(fileUrl);
            message = "downloading>>" + fileUrl;
            facade.SendMessageCommand(NotiConst.UPDATE_MESSAGE, message);
            /*
            www = new WWW(fileUrl); yield return www;
            if (www.error != null) {
                OnUpdateFailed(path);   //
                yield break;
            }
            File.WriteAllBytes(localfile, www.bytes);
             */
            //这里都是资源文件，用线程下载
            BeginDownload(fileUrl, localfile);
            while (!(IsDownOK(localfile))) { yield return new WaitForEndOfFrame(); }
        }
    }
    yield return new WaitForEndOfFrame();

    message = "更新完成!!";
    facade.SendMessageCommand(NotiConst.UPDATE_MESSAGE, message);

    OnResourceInited();
}
```
1. 通过Packager.cs将lua脚本,资源打包成ab,每个资源生成对应的md5值保存在files.txt中
2. 将新生成的ab和files.txt放入web服务器的下载目录中
3. 游戏启动,从web服务器中下载files.txt与游戏中的files.txt进行对比,如有资源的md5值不同,则该资源需要更新
4. 下载更新完成，调用OnResourceInited函数,将ab资源初始化，启动lua虚拟机,游戏开始
