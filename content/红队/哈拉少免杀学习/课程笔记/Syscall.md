# 课堂随记
1. 对抗一些免费的杀软用不用syscall都是差不多可以的，通过一些不常用的API函数调用再加一些规避手段绕过国内的AV基本是可以的了
2. 使用syscall可以提高规避能力，同时也增加了一些检测点
3. 360核晶做的是一个r0的hook就是关于SSDT的hook，但是国外EDR由于有规定所以不能做内核层的hook所以基本都是R3的hook
4. EDR通常会将自己的aswhook.dll文件注入进程去hook常用dll中的函数，包括ntdll.dll中
5. 测试hook的话推荐去安装avast或者bitdefender，avast免费，另一个好像被墙掉了
6. 规避hook整体上有三大类方法：unhook，直接/间接系统调用，自实现R3函数
7. EDR通常会将自己的监控DLL注入到每个新进程中
8. 缺点：syscall中可能包含syscall这几个特征字符串；EDR现在开始检测调用是否从ntdll中发起，只要不是就认为是直接系统调用；
9. 直接系统调用对抗火绒，360核晶可以，对抗WDF不行
10. vs写syscall是会出现要添加汇编文件作为依赖项等设置，但是使用过Clion+mingw编译就没事
11. 国外的EDR上线很简单，想要做行为上的规避可以使用其他杀软自身的白加黑或者bof内存加载

# Syscallwh
## 使用方法
1. 利用github上的教程直接克隆使用，会生成两个文件一个是.ASM 另一个是.H文件，使用两个文件的方式在自己的代码中包含一下
2. 其中.h文件包含了调用之后的声明的函数原型
3. syswh2已经加入间接系统调用的功能了，syswh3和2项目相差不大
4. <mark style="background: #FF5582A6;">对于国内的AV/EDR来讲，只要CS +Profile流量做好，loader能过掉静态成功上线，最后就是注意一些关于opsec的操作规范就好</mark>
5. 像天勤这种EDR对流量的审查会很严格，所以不能只使用profile进行规避，最好加一些其他的方法
6. 间接系统调用项目推荐：https://github.com/Maldev-Academy/HellHall

# Unhook
1. unhook最致命点就是：特征明显，特别明显，会有二次加载ntdll的痕迹
2. 