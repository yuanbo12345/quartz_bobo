# 堆栈欺骗技术深度解析：静态与动态方法

## 零、前言：为何要欺骗堆栈？

在现代攻防对抗中，EDR（端点检测与响应）和安全分析师会重点监控线程的**调用堆栈（Call Stack）**，尤其是在程序执行敏感操作或长时间“睡眠”（Sleep）时。调用堆栈就像一个“任务执行记录本”，它清晰地记录了函数之间的调用关系。

一个未经处理的恶意软件，其调用堆栈会暴露它的意图：

```
[栈顶]
ntdll.dll!NtDelayExecution  <-- Sleep函数的最终系统调用
KERNELBASE.dll!SleepEx
MyMaliciousImplant.exe!BeaconingFunction  <-- 恶意软件的信标函数
MyMaliciousImplant.exe!main
[栈底]
```

EDR看到这个堆栈，会立刻识别出`BeaconingFunction`是可疑的，从而触发警报。

**堆栈欺骗（Stack Spoofing）** 的核心目标就是伪造这个“记录本”，当EDR来检查时，让它看到一个看起来完全无害的调用路径，从而隐藏我们真正的恶意行为。

---

## 一、静态堆栈欺骗 (Static Stack Spoofing)

静态方法的核心是**凭空捏造**一个全新的、虚假的调用堆栈。

### 1.1 技术原理：狸猫换太子

我们并不会修改当前线程正在使用的真实堆栈，而是：
1.  **另起炉灶**：在内存的其他地方（如堆区）申请一块新内存。
2.  **伪造现场**：在这块新内存上，手动地、一个一个地写入我们希望EDR看到的“返回地址”，精心打造一个看起来天衣无缝的假堆栈。这些地址通常来自无害的系统DLL。
3.  **偷天换日**：在调用目标函数（如`Sleep`）前的一瞬间，通过一小段汇编代码，强行将CPU的**堆栈指针寄存器（RSP）** 从指向真实堆栈，改为指向我们伪造的假堆栈。
4.  **执行与恢复**：CPU使用假堆栈执行目标函数。函数返回后，我们再通过汇编将RSP指针换回真实的堆栈，程序继续正常执行。

### 1.2 技术流程
1.  **准备“跳板”函数 (Gadget)**：创建一个极简的函数，其唯一作用就是调用真正的`Sleep`。这将是假堆acks上的最后一站。
2.  **申请内存**：使用`VirtualAlloc`在堆上为假堆栈申请一块内存。
3.  **构建假堆栈**：从申请内存的高地址开始，向低地址依次“压入”伪造的返回地址和“跳板”函数的地址。
4.  **切换并调用**：编写内联汇编，完成以下操作：
    a. 备份当前的`RSP`。
    b. 将`RSP`指向假堆栈。
    c. `call`位于假堆栈顶的“跳板”函数。
    d. 调用结束后，恢复`RSP`。
5.  **清理现场**：使用`VirtualFree`释放申请的内存。

### 1.3 简要示例代码

*注意：此为原理性展示，x64下的内联汇编在不同编译器（MSVC/GCC）中语法不同且存在限制。*
```cpp
#include <windows.h>

// 跳板函数，必须防止内联优化
__declspec(noinline)
void SleepGadget(DWORD milliseconds) {
    Sleep(milliseconds);
}

// 自定义的静态欺骗Sleep
void StaticSleep(DWORD milliseconds) {
    // 1. 申请假堆栈内存
    const size_t fakeStackSize = 1024;
    void* fakeStack = VirtualAlloc(NULL, fakeStackSize, MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);
    if (!fakeStack) return;

    // 2. 获取一个无害的返回地址用于伪造
    void* fakeReturnAddress = (void*)GetProcAddress(LoadLibraryA("user32.dll"), "GetForegroundWindow");

    // 3. 构建假堆栈 (从高地址到低地址)
    void** pFakeStack = (void**)((char*)fakeStack + fakeStackSize);
    *--pFakeStack = fakeReturnAddress; // 压入一个伪造的返回地址
    *--pFakeStack = (void*)SleepGadget; // 压入跳板函数地址

    // 4. 切换堆栈并调用 (GCC/Clang风格汇编)
    void* oldRsp;
    __asm__ __volatile__(
        "mov %%rsp, %0\n\t"   // 备份真实RSP
        "mov %1, %%rsp\n\t"   // 切换到假RSP
        "sub $32, %%rsp\n\t"  // 为call腾出影子空间 (x64调用约定)
        "call *%2\n\t"        // 调用栈顶的函数 (SleepGadget)
        "mov %0, %%rsp"       // 恢复真实RSP
        : "=r"(oldRsp)
        : "r"(pFakeStack), "r"(*(pFakeStack)) 
        : "memory"
    );

    // 5. 清理
    VirtualFree(fakeStack, 0, MEM_RELEASE);
}
```

---

## 二、动态堆栈欺骗 (Dynamic Stack Spoofing)

动态方法是静态方法的进阶，它不凭空捏造，而是**篡改历史**，生成的假堆栈更加逼真。

### 2.1 技术原理：清洗真实记录

动态欺骗不再创建完全虚构的堆栈，而是：
1.  **复制真实堆栈**：在调用`Sleep`之前，先把当前线程**完整、真实**的调用堆栈内存复制一份出来。
2.  **清洗敏感记录**：在这份**副本**上，从上到下（从栈顶到栈底）检查每一个返回地址。如果发现某个返回地址指向我们自己的恶意模块，就用一个无害的、来自系统DLL的函数地址（Gadget）去**覆盖**它。
3.  **切换并执行**：将堆栈指针`RSP`指向这个被“清洗”过的堆栈副本，然后调用`Sleep`。

**优势**：伪造的堆栈90%的内容都是真实的，它保留了程序真实的调用路径，只是把其中最关键的、指向我们自己的那一部分给“抹掉”了。这使得伪造的堆栈看起来极为逼真。

### 2.2 技术流程
1.  **确定真实堆栈边界**：通过`RSP`寄存器和线程环境块（TEB）找到当前栈的精确起止地址。
2.  **复制真实堆栈**：计算堆栈大小，使用`VirtualAlloc`申请同样大小的内存，并通过`memcpy`完成完整复制。
3.  **清洗堆栈副本**：
    a. 遍历堆栈副本。
    b. 检查每一个栈上的值是否是一个指向代码段的指针（返回地址）。
    c. 如果是返回地址，再判断它是否指向我们自己的恶意模块。
    d. 如果是，就从预先准备好的“无害地址列表”中随机挑选一个进行替换。
4.  **准备跳转**：在清洗过的堆栈副本的顶部，压入“跳板”函数（`SleepGadget`）的地址。
5.  **切换并调用**：使用与静态方法相同的汇编技巧，切换`RSP`，调用`SleepGadget`，然后恢复`RSP`。
6.  **清理现场**：释放为堆栈副本申请的内存。

### 2.3 简要示例伪代码

由于完整实现非常复杂，这里使用伪代码展示核心逻辑。
```cpp
void DynamicSleep(DWORD milliseconds) {
    // 1. 获取真实堆栈边界
    void* stackTop = GetCurrentStackTop(); // 通过RSP寄存器
    void* stackBase = GetCurrentStackBase(); // 通过TEB
    size_t stackSize = (char*)stackBase - (char*)stackTop;

    // 2. 复制真实堆栈
    void* fakeStack = VirtualAlloc(NULL, stackSize, ...);
    memcpy(fakeStack, stackTop, stackSize);

    // 3. 清洗堆栈副本
    void* ourModuleBase = GetModuleHandle(NULL);
    // ... 获取模块大小 ...

    // 遍历副本，查找并替换指向我们自己模块的返回地址
    for (size_t i = 0; i < stackSize; i += sizeof(void*)) {
        void** pAddress = (void**)((char*)fakeStack + i);
        if (IsReturnAddressToOurModule(*pAddress, ourModuleBase)) {
            *pAddress = GetRandomBenignAddressFromSystemDLL();
        }
    }

    // 4. 准备并执行跳转 (与静态方法类似)
    // ... 计算新栈顶，压入跳板函数 ...
    
    // 5. 使用汇编切换RSP，调用，然后恢复
    SwitchStackAndCall(...);

    // 6. 清理
    VirtualFree(fakeStack, 0, MEM_RELEASE);
}
```

---

## 三、静态 vs. 动态：对比总结

| 特性 | 静态堆栈欺骗 | 动态堆栈欺骗 |
| :--- | :--- | :--- |
| **核心思想** | 凭空捏造 (Fabrication) | 篡改历史 (Tampering) |
| **真实性** | 较低，依赖于伪造的精细程度 | 非常高，保留了大部分真实调用链 |
| **实现复杂度** | 相对简单 | 非常复杂，涉及TEB、堆栈帧遍历等 |
| **性能开销** | 较小 | 较大（需要复制整个堆栈并遍历） |
| **适用场景** | 简单的、调用层级不深的环境 | 复杂的、需要高仿真度对抗的环境 |

**结论**：堆栈欺骗是现代恶意软件和红队工具中用于规避检测的重要技术。从相对简单的静态方法到高度逼真的动态方法，其演进反映了攻防双方技术的不断升级。这些技术的研究有助于蓝队更好地理解威胁并开发更先进的检测策略。

```