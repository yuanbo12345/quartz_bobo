## 执行策略
1. 基本上就是在ps的脚本上加一个bypass命令以改变执行策略，实现所有恶意命令都可以执行
2. 360想要pws上线只能利用其他的方式，不能直接使用命令行执行的方式，360直接监视的就是命令行软件
## pws常用混淆脚本工具
1. 最推荐：https://github.com/danielbohannon/Invoke-Obfuscation，旧工具只用这个，自己一定要找一些新的工具，寻找关键词：powershell obfuscator找最新的
2. 可以直接使用cs生成的pws脚本，然后混淆

## 将pws转换成exe文件执行
1. 第一种利用：ps2exe.ps1去转换
2. 第二种利用：一个图形化工具去转win-ps2exe或者是ps1_to_exe工具，第二个工具比较好用
3. 重点注意：那种混淆甚至是转成exe也是一样，本质上最后执行的还是pws脚本，所以一定要能够bypass amsi才行

## 利用C# 实现pws的一些功能
1. C#可以内存加载
2. 利用vs编译的话需要将系统的dll添加上去，位置![[1746781902189.jpg]]关键dll就是system.management.automation.dll
3. 编写示例![[1746782057512.jpg]]
### 代码解释
1. 创建管道的作用是为了回显
2. 相关提供的脚本就是我们自己的恶意pws脚本
3. 