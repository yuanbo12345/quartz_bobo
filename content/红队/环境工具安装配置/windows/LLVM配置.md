1. 首先就是在vs上面安装好clang和llvm，进入vstudio installer中安装
2. 接着下载llvm下载地址：[发布 ·Backengineering/LLVM-MSVC （英语） --- Releases · backengineering/llvm-msvc](https://github.com/backengineering/llvm-msvc/releases)
3. 下载：![[image-39.png]]
4. 环境是windows x86_64 cmake mingw（上面得项目估计是已经编译好的了，所以可能并不是需要cmake和mingw了）
5. 安装下面得exe文件，解压windows--llvm压缩包将其中得所有文件替换到llvm安装之后生成得文件夹的bin文件夹中
6. 注意：配置VS的时候一定要重新创建一个项目去配置，如果重新生成exe过程中显示代码报错，可以试一下更换llvm版本以及更换windows SDK版本测试一下
7. 混淆命令：`-mllvm -string-obfus -mllvm -const-obfus -mllvm -fla -mllvm -sub_loop=2 -mllvm -bcf_loop=1 -mllvm -bcf_prob=10 `
8. 


## 其他项目尝试
### 已经编译好的
1. 