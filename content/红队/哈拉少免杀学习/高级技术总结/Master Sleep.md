# 基于系统对象的阻塞式休眠(<mark style="background: #FF5582A6;">重点</mark>)
- **NtWaitForSingleObject()**、**NtWaitForMultipleObjects()**
- 伪造或创建**匿名事件 / section / job / semaphore**，调用 `NtWaitForSingleObject` 等阻塞等待，实际休眠，且逃避 EDR 对 `Sleep` 的 hook 检测。
举例
```
HANDLE hEvent = CreateEvent(NULL, TRUE, FALSE, NULL);
NtWaitForSingleObject(hEvent, FALSE, &timeout)；
```
逃避主流 EDR inline hook，且行为看似系统阻塞等待，像正常进程逻辑

1. 用 `NtWaitForSingleObject` + 匿名 Event，纯原生 NTAPI 调用，**无导出 Sleep/NtDelayExecution**
2. 实战基本采用这种方法，示例代码如下
3. 使用方法：调用形式`StealthSleep(3000); // 休眠3000秒`
```cpp
#include <Windows.h>
#include <winternl.h>

// 定义 NtWaitForSingleObject 原型
typedef NTSTATUS (NTAPI* pNtWaitForSingleObject)(
    HANDLE Handle,
    BOOLEAN Alertable,
    PLARGE_INTEGER Timeout
);

void StealthSleep(DWORD milliseconds)
{
    HMODULE hNtdll = GetModuleHandleW(L"ntdll.dll");
    if (!hNtdll) return;

    pNtWaitForSingleObject NtWaitForSingleObject = (pNtWaitForSingleObject)GetProcAddress(hNtdll, "NtWaitForSingleObject");
    if (!NtWaitForSingleObject) return;

    HANDLE hEvent = CreateEventW(NULL, TRUE, FALSE, NULL);
    if (!hEvent) return;

    LARGE_INTEGER liDueTime;
    liDueTime.QuadPart = -1 * (LONGLONG)milliseconds * 10000;

    NtWaitForSingleObject(hEvent, FALSE, &liDueTime);

    CloseHandle(hEvent);
}
```
## 休眠原理
- `NtWaitForSingleObject` 阻塞线程
- Event 永不触发，等超时自动唤醒
- 线程状态是 `Wait`，系统调度器接管
- 卡巴内存扫描扫不到 Beacon / Shellcode 段
## Loader中存放位置
1. 一般在shellcode即将要加载的时候调用


# TimerQueue + Callback 延迟执行
- 注册一个 `CreateTimerQueueTimer` 定时器，执行回调   
- 然后阻塞当前线程

**优点**：TimerQueue Callback 很少被 EDR 检查，特别是像 Cobalt Strike 就喜欢这样调度 C2 beacon。


# NtAlertThread/NtTestAlert + Alertable Wait
- 将线程置于 alertable 状态，利用 `NtDelayExecution(TRUE)` 或 `NtWaitForSingleObject(TRUE)`
- 或者主动向自己线程投递 APC，在线程中断时恢复执行
**优点**：EDR 极少检测 NtTestAlert 和 NtAlertThread 调用链。

