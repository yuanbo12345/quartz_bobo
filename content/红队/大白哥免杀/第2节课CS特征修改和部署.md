## 课堂疑问
1. 利用shell软件和linux服务器建立通信的过程是ssh协议吗？
2. 搞清cs的整个通信过程怎么监听，开放哪个段口？
3. cs下发命令，shellcode回连等各种通信都是什么协议流量？
4. 由于后渗透阶段任然需要下新的木马（不同功能），所以我们常规使用的stagerless是指后渗透时也不能继续下发木马，还是说只是第一阶段的shellcode采用远程下载的方式加载到受害者主机上？
5. 


## 修改CS的特征
1. 修改特征只是针对CS的服务端修改，客户端不进行修改
### 修改CS的默认端口

### 修改证书
1. 主要目的就是修改cs中的一些常被杀软标记的特征字符
2. 推荐使用正规证书（自己购买的付费证书）
3. 自签名证书是自己生成的伪造证书
4. 使用`mv`命令备份原始的ssl证书文件时就是给原始文件重命名保存了一下
#### 生成证书的命令参数详解
```shell
keytool -keystore cobaltstrike_new.store -storepass 1qazwsx -keypass 1qazwsx -genkey -keyalg RSA -alias qq.com -dname "CN=US, OU=qq.com, O=Software, L=Somewhere,ST=Cyberspace, C=CN"
```
1. `-keystore`参数代表新生成证书的名字
2. `-storepass`参数表示新生成的证书密码（就是证书默认口令）
3. `-keypass`参数表示再确认证书密码
4. `-genkey -keyalg`参数表示
5. `-alias`参数表示的是域名
6. `-dname`参数表示的是基于域名的一些信息
7. 由于CS的证书文件已经再team上写死了，所以证书文件名也要和原先保持一致
<mark style="background: #FF5582A6;">最后生成证书重命名之后，不要忘记要在team文件中修改密码成新生成证书的密码</mark>

注意：除了jks格式的证书其他格式的证书都需要转换格式，转换方法在课程的原笔记上


## 修改Profile文件
<mark style="background: #FF5582A6;">目标：就是拿到网上公开的prroflie文件之后去修改好就行</mark>

### 课堂随记
1. defender或者卡巴对内存查杀比较厉害，由于后渗透之后还需要进行stager的下发以运行其他渗透功能，所以我们要对之后阶段的stager进行免杀，通过更改profile上的二阶段stager代码块就可以实现简单的初级内存免杀

### 通过工具修改proflie以生成垃圾代码+shellcode
1. 工具名：prepend.py
2. 通过命令：python3 prepend.py运行
3. 然后将生成的shellcode代码复制粘贴到配置文件中（在 transform-x64 或 transform-x86 块内
4. 工具：rich_header.py也很重要
5. 原生笔记上关于检查profile上最后的补充，是关于github上找开源的最新profile的网址，方法之类的

<mark style="background: #FF5582A6;">注意：修改完成之后一定要去检查profile是否正常，只看中括号的就行，红色是报错，黄的是警告</mark>


## CS配置上线Linux主机
1. 看原生笔记最好
2. 编辑文件时，改的是path和bin两处地方，分别改成自己下载的插件的目录和插件执行的程序名（注意，路径最后一定不要忘记加两个\\）
3. 注意给客户端的java加入环境变量
4. cat版本cs是github上的开源二开cs的项目可以看一下（建议熟悉原生的cs之后再去使用二开的）

