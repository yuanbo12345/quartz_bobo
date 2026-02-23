# UAC原理
1. uac=提权
2. 管理员权限的SID是500
## 触发uac的条件
1. ![[1746784027907.jpg]]
2. 触发之后的流程![[1746784117380.jpg]]
3. 所以bypassuac本质就是过掉uac的弹框，然后给<mark style="background: #FF5582A6;">进程提升到管理员权限</mark>
## Bypass UAC方法
1. 主要方法如下![[1746784946167.jpg]]
2. 最常用的就是利用白名单
3. 计划任务用的不多
4. 利用dll劫持，大部分方法就是将黑dll移动到高权限文件夹，但是大部分情况是拒绝访问
5. 关于过uac的一些开源方法整合项目：https://github.com/hfiref0x/UACME
6. 白名单过uac最致命的确定就是要动注册表针对无杀软环境最好
7. 现在实战最主要用com接口，com绕过uac首推利用c#程序，应为c#程序调用com接口很简单
8. 其次就是利用RPC SSPI很稳定上线之后基本上就是一个system权限
## sspi过uac
1. 利用的现有项目：https://github.com/antonioCoco/SspiUacBypass
2. 要改一些地方：![[1746785552543.jpg]]
3. 570行代码，360肯定会去拦截cmd启动这种，<mark style="background: #FF5582A6;">直接去掉cmd/c就行</mark>
4. 有一个用于havoc的bof项目，也是改掉cmd/c，写成cs的bof形式使用，项目地址：https://github.com/icyguider/UAC-BOF-Bonanza就是些个cna文件就可以
5. 通过sspi去启动自己的上线马存活时间只能有60秒，所以要赶紧将进程spwan出去
6. 第二种方法就是利用cmd/c执行，但是可以将cmd换成一个lolbin替代执行，叫做for.exe的间接执行自己的上线马
7. 一定要多看UACME，并进行测试，测试方式：直接内存加载uacme的exe文件，上线之后去测试，dll劫持动注册表的这些直接不用看，肯定会被给拦截，重点关注利用COM接口的
8. 注意：灰进程执行com组件passuac都会拦截，但是白加黑的形式基本放行
9. 测试uacme的时候可以将项目编译之后，直接开始测试，利用pws窗口执行相关命令，然后直到这个方法可以打开某一个pe文件，就可以单独把这个方法拿出来，看代码自己写相关的实现用法。推荐测试时执行C:\winodws\system32\notepad.exe这个可以被加载出来，就可以把这段代码单独拿出来做免杀使用，记住测试时一定要将免杀先关闭

### 利用COM组件BypassUAC
1. com组件本质上是系统中的pe文件
2. CLSID![[1746889002098.jpg]]
3. <mark style="background: #FF5582A6;">重点关注</mark>：如果我们在本进程使用bypassuac，那么系统一定会弹框显示；如果是带有合法签名的白程序（最好是微软的）那么就不会，所以利用com接口提权最好的方式就是白加黑
4. 有些com组件的接口还具备命令执行，创建指定进程的功能，所以我们可以利用com组件去提升权限的同时执行我们的相关pe文件
5. 什么样的com组件才能被利用来绕过uac![[1746889465803.jpg]]
6. 可以利用工具查看com接口的一些属性信息：https://github.com/tyranid/oleviewdotnet
7. com接口利用的代码编写![[1746890360077.jpg]]
8. 如果落地exe感觉很困难，可以用bof：https://github.com/tijme/cmstplua-uac-bypass，没有核晶的时候实战用这个最好了，有核晶就用sspi那个项目
9. 头像哥：https://www.zcgonvh.com/
10. 