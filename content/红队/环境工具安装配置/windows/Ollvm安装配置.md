https://github.com/DreamSoule/ollvm17

直接使用这个项目的最新发行版即可，这个是已经预编译好的了![[image-235.png]]
上图就是下载这个就可以，预编译好了的

1. 下载完成后直接解压就可以![[image-236.png]]
2. 如图，箭头指向的就是利用这个编译，可以先加入系统环境变量
3. 可以采用命令行编译，但是一定要使用![[image-237.png]]这个命令行工具
4. 编译的时候加上混淆参数就可以，参数可以从项目网站上查看即可
5. 如果编译的时候有问题，可以<mark style="background: #FF5582A6;">使用clang的绝对路径编译</mark>，因为你可能环境变量中有两个clang编译器，命令行不知道使用哪一个了

## 方法二就是在visualstudio中直接编译
采用其他项目完成
主要教程网站：https://foxi.buduanwang.vip/virtualization/pve/3195.html/
1. 首先下载安装官方llvm包：下载地址：https://github.com/llvm/llvm-project/releases/tag/llvmorg-17.0.6
2. ![[image-239.png]]
3. 使用![[image-240.png]]这个项目中的解压出来文件中的几个clang开头的文件替换到官方原生的bin文件中，替换
4. ![[image-241.png]]如果属性管理器找不到，搜索一下转到，回到项目管理器时点击属性管理器右上角的叉，多点几下就行
```props
<Project>
   <PropertyGroup>
      <LLVMInstallDir>D:\Ollvm\soft\LLVM</LLVMInstallDir>
      <LLVMToolsVersion>17.0.6</LLVMToolsVersion>
   </PropertyGroup>
</Project>
```
命名为：`Directory.build.props`，放在固定位置，每次ollvm混淆的时候就添加这个可以
注意：上述代码中的项目地址要换成自己的

![[image-242.png]]


常用混淆命令
```bash
-mllvm -enable-bcfobf -mllvm -enable-cffobf -enable-funcwra -enable-strcry 
```
