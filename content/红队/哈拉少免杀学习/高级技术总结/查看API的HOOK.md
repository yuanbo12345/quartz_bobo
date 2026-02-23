# 方法一
## 利用进程查看其加载的dll
1. 先编写一个最简单的示例loader代码，编译成release之后运行，通过process hacker查看加载的dll有没有AV/EDR的。
2. 基础测试loader示例代码如下，下面的代码只能检查一个敏感API的hook
3. 测试代码编写要求：<mark style="background: #FF5582A6;">帮我写一个简单的能运行的程序，最好一直运行那种，利用用到一些敏感的api函数调用，在defender下不报毒，我想测试一下是否被hook了，用c++编写</mark>

```cpp
// hookTest.c  
#include <windows.h>  
#include <stdio.h>  
  
int main(int argc, char const *argv[])  
{  
char x;  
scanf("%c", &x);  
printf("%c", x);  
LPVOID test = VirtualAllocEx(NULL, NULL, sizeof(x) * 4, MEM_COMMIT, PAGE_READWRITE);  
return 0;  
}
```

加载的dll展示如下
![[image-76.png]]

## 继续利用X64 DBG设置硬件断点
1. 找到常见的API函数，然后设置上断点以跟踪
2. 接着看一下干净的API汇编语句和开启AV/EDR之后的汇编语言
3. 看一下被挂钩之后的函数的字节变换，然后据此编写查看代码
4. 下面的示例代码：是在发现bitdefender对函数第一个字节就变成jmp函数之后编写的
```cpp
// listAllHooked.c  
#include <windows.h>  
#include <stdio.h>  
char *dbghelp_lst[] = {"EnumerateLoadedModules"};  
char *kernel32_lst[] = {"CloseHandle",  
"CloseHandle",  
"ConvertThreadToFiber",  
"CreateFiber",  
"CreateRemoteThreadEx",  
"CreateThread",  
"EnumSystemLocalesA",  
"GetCurrentProcess",  
"GetCurrentThread",  
"HeapAlloc",  
"HeapCreate",  
"OpenProcess",  
"QueueUserAPC",  
"ReadProcessMemory",  
"SwitchToFiber",  
"VirtualAlloc",  
"VirtualAllocEx",  
"VirtualProtect",  
"VirtualProtectEx",  
"WaitForSingleObject",  
  
"WriteProcessMemory"};  
char *ntdll_lst[] = {"EtwpCreateEtwThread",  
"NtQueryInformationProcess",  
"NtQueueApcThreadEx",  
"RtlCopyMemory",  
"RtlCreateUserThread"};  
char *rpcrt4_lst[] = {"UuidFromStringA"};  
void check_hooked(char *moduleName, char **api_lst)  
{  
  
// for (int i = 0; i < sizeof(api_lst) / sizeof(api_lst[0]); i++)  
printf("[+] Test hooking for %s \n", moduleName);  
int i = -1;  
while (api_lst[++i] != NULL)  
{  
  
HANDLE kernalbase_handle = GetModuleHandle(moduleName);  
char *curr_name = api_lst[i];  
LPVOID CRT_address = GetProcAddress(kernalbase_handle, curr_name);  
if ((int)CRT_address != 0)  
{  
byte curr_value = (byte) * (char *)CRT_address;  
// sprintf(curr_value, "%02X \n", (byte) * (char *)CRT_address);  
// byte jmp_opcode = (byt"\xE9"e)"\xE9";  
if (curr_value == 0xe9)  
{  
printf("%s at %p : ", curr_name, CRT_address);  
printf("0x%02X \n", (byte) * (char *)CRT_address);  
}  
// printf("error in %s \n", curr_name);  
}  
  
/* code */  
}  
}  
  
int main(int argc, char const *argv[])  
{  
check_hooked("dbghelp", dbghelp_lst);  
check_hooked("kernel32", kernel32_lst);  
check_hooked("ntdll", ntdll_lst);  
check_hooked("rpcrt4", rpcrt4_lst);  
  
printf("[+] Done \n");  
char x;  
scanf("%c", &x);  
printf("%c", x);  
  
return 0;  
}
```
