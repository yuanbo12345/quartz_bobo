分析网站来源：https://www.esentire.com/blog/pure-crypter-malware-analysis-99-problems-but-detection-aint-one
# 规避方法分析
## 执行任意Pws命令
1. ![[image-56.png|612x610]]


## 持久化处理
1. 下面的方法处理持久化，持久化是通过将加载器写入配置中指定的目录，然后通过运行键、计划任务或 VBScript (.vbs) 在用户的启动文件夹中设置持久化来实现的。
2. ![[image-57.png]]
3. ![[image-58.png]]


## 反文件删除功能
1. 下一个可选方法对应于 Pure Crypter 中的“反文件删除”功能。这种技术涉及以仅 FILE_SHARE_READ 权限打开文件句柄，并通过 Windows API DuplicateHandle()让 explorer.exe 接收该句柄。
2. ![[image-59.png]]


## 父进程欺骗
1. ![[image-60.png]]
2. ![[image-61.png]]

## 联网阻止
1. 下一个可选方法通过 LOLBin 中的 ipconfig.exe 禁用互联网。此方法的目的在于阻止 AV/EDR 与其后端通信，从而阻止上传样本进行分析。
2. ![[image-62.png]]


## 执行延迟睡眠
1. 执行延迟功能利用 Windows API SleepEx 来等待指定秒数。
2. ![[image-63.png]]


## 禁用AMSI
### 方法一
1. ![[image-64.png]]
2. ![[image-65.png]]


### 方法二
1. ![[image-66.png]]
2. ![[image-67.png]]


## DLL Unhook
1.  Pure Crypter 中的“Dll Unhooking”功能，它用于加载“干净”的 kernel32.dll（Windows 10 或更高版本）和 ntdll.dll 副本，从而有效地绕过任何已由 AV/EDR 设置的挂钩。
2. ![[image-68.png]]
