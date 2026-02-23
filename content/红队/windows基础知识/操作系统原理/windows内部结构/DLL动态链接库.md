1--- DLL 描述为“一个包含可由多个程序同时使用的代码和数据的库”。本质上就是一类专用的函数，方法或者数据，可以被其他程序随意调用，只是起完整作用代码的一部分

2---当 DLL 作为程序中的函数加载时，该 DLL 被指定为依赖项。由于程序依赖于 DLL，因此攻击者可以瞄准 DLL 而不是应用程序来控制执行或功能的某些方面。也就是说此时dll可以被作为一个恶意的函数被其他代码调用以执行其函数中的恶意功能。

### DLL的代码开发
1---dll也是由c++语言编写而成

2---DLL的头文件；它将定义导入和导出哪些函数。
```c++
#include "stdafx.h"
#define EXPORTING_DLL
#include "sampleDLL.h"
BOOL APIENTRY DllMain( HANDLE hModule, DWORD ul_reason_for_call, LPVOID lpReserved)
{
    return TRUE;
}

void HelloWorld()
{
    MessageBox( NULL, TEXT("Hello World"), TEXT("In a DLL"), MB_OK);
}
```
上述为dll的示例代码

<mark style="background: #FF5582A6;">可以关注到dll有一个主函数名字为dllmain( )，下面又定义了一个函数，说明dll可以写成自己实现完整功能的代码</mark>

### 应用程序使用DLL
DLL 有两种使用的方式：加载时动态链接或运行时动态链接以将自己加载到程序中

#### 加载时动态链接
应用程序将显式调用 DLL 函数。您只能通过提供标头 (.h) 和导入库 (.lib) 文件来实现这种类型的链接。下面是从应用程序调用导入的 DLL 函数的示例。
```cpp
#include "stdafx.h"
#include "sampleDLL.h"
int APIENTRY WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow)
{
    HelloWorld();
    return 0;
}
```


#### 运行时动态链接
运行时动态链接加载时，将使用单独的函数（ `LoadLibrary` 或 `LoadLibraryEx` ）在运行时加载 DLL。加载后，您需要使用 `GetProcAddress` 来标识要调用的导入DLL函数。下面是在应用程序中加载和导入 DLL 函数的示例。
```cpp
typedef VOID (*DLLPROC) (LPTSTR);
...
HINSTANCE hinstDLL;
DLLPROC HelloWorld;
BOOL fFreeDLL;

hinstDLL = LoadLibrary("sampleDLL.dll");
if (hinstDLL != NULL)
{
    HelloWorld = (DLLPROC) GetProcAddress(hinstDLL, "HelloWorld");
    if (HelloWorld != NULL)
        (HelloWorld);
    fFreeDLL = FreeLibrary(hinstDLL);
}
```
<mark style="background: #FF5582A6;">在恶意代码中，威胁行为者通常会更多地使用运行时动态链接而不是加载时动态链接。这是因为恶意程序可能需要在内存区域之间传输文件，并且传输单个 DLL 比使用其他文件要求进行导入更易于管理。</mark>

#### DLL的调用执行（重要知识）
**DLL（动态链接库）本身不能被独立执行**，它需要一个进程（EXE 或其他加载器）来调用它的函数。但在某些特殊情况下，可以**间接或变通地让 DLL 执行代码**。

##### 1. 普通情况下，DLL 不能独立运行

- DLL 文件（.dll）本质上是一个**共享库**，必须由其他可执行程序（如 .exe）调用，不能像 EXE 那样直接运行。
- 当没有进程加载 DLL 时，操作系统不会主动运行它

##### 2.特殊情况下，DLL可以“自主执行”
虽然 DLL 不能独立运行，但有一些**变通方法**可以让 DLL 代码执行：
###### 使用 rundll32.exe 运行 DLL
但这种方式**要求 DLL 具有特定的导出函数格式**，否则会导致崩溃或报错。
**这种方法一般都使用PS，VBS，VBH脚本去运行rundll.32程序**

###### DLL 劫持 / 反射加载
一些高级技术允许 DLL 在不依赖 EXE 进程的情况下执行（本质也是靠其他程序调用）：

- **DLL 劫持（DLL Hijacking）**：通过放置恶意 DLL 在应用程序搜索路径中，使合法程序误加载它。
- **反射加载（Reflective DLL Injection）**：在内存中手动加载 DLL 并执行代码，不需要注册到系统。（利用rundll.32和脚本的注册函数）

###### 将 DLL 重新封装为 EXE 
如果需要让 DLL 变得可执行，可以创建一个小型的 EXE 来调用它，比如：

- **写一个简单的 C/C++ 代码调用 LoadLibrary()**
- **使用 Python + ctypes 加载 DLL**
- **利用 PowerShell 调用 DLL**

