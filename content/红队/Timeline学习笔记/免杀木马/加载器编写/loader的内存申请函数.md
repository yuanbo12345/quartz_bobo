# 加载器编写套路
1--申请内存
2--将shellcode复制到内存中
3--执行shellcode
## 加载器重要函数
![[e9a03395786a20644c3be405c4980ab.jpg]]

# 申请内存套路
## `VirtualAllox`+`VirtualProtect`的api利用
### `VirtualAllox`函数的介绍示例
`VirtualAlloc` 用于在调用进程的虚拟地址空间中保留或提交内存区域。
函数原型
```c++
LPVOID VirtualAlloc(
  LPVOID lpAddress,       // 指定要分配的内存区域的起始地址（通常为 NULL，由系统自动选择）
  SIZE_T dwSize,          // 要分配的内存区域的大小（以字节为单位）
  DWORD  flAllocationType, // 分配类型（如 MEM_COMMIT 或 MEM_RESERVE）
  DWORD  flProtect         // 内存保护选项（如 PAGE_READWRITE）
);
```
#### 参数说明
- **`lpAddress`**：
    - 指定要分配的内存区域的起始地址。
    - 如果为 `NULL`，系统会自动选择一个合适的地址。  
- **`dwSize`**：
    - 要分配的内存区域的大小（以字节为单位）。
- **`flAllocationType`**：
    - 分配类型，常用的选项有：
        - `MEM_COMMIT`：提交物理内存或页面文件。
        - `MEM_RESERVE`：保留虚拟地址空间，但不提交物理内存。    
- **`flProtect`**：
    - 内存保护选项，常用的选项有：  
        - `PAGE_READWRITE`：可读可写。      
        - `PAGE_EXECUTE_READWRITE`：可执行、可读、可写。        
#### 返回值
- 如果成功，返回分配的内存区域的基地址。
- 如果失败，返回 `NULL`。


### `VirtualProtect`函数
`VirtualProtect` 用于更改调用进程的虚拟地址空间中已提交内存区域的保护属性。
函数原型
```c++
BOOL VirtualProtect(
  LPVOID lpAddress,       // 要更改保护属性的内存区域的起始地址
  SIZE_T dwSize,          // 要更改保护属性的内存区域的大小（以字节为单位）
  DWORD  flNewProtect,    // 新的内存保护选项
  PDWORD lpflOldProtect   // 用于保存旧的内存保护选项
);
```
#### 参数说明
- **`lpAddress`**：
    - 要更改保护属性的内存区域的起始地址。
- **`dwSize`**：
    - 要更改保护属性的内存区域的大小（以字节为单位）。   
- **`flNewProtect`**：
    - 新的内存保护选项，常用的选项有：  
        - `PAGE_EXECUTE_READWRITE`：可执行、可读、可写。    
        - `PAGE_READWRITE`：可读可写。     
- **`lpflOldProtect`**：
    - 用于保存旧的内存保护选项。    
#### 返回值
- 如果成功，返回 `TRUE`。
- 如果失败，返回 `FALSE`。

### 两者综合示例
```c++
#include <windows.h>
#include <iostream>

int main() {
    // 申请内存
    LPVOID pMemory = VirtualAlloc(
        NULL,                   // 由系统自动选择地址
        4096,                   // 分配 4096 字节（1 页）
        MEM_COMMIT | MEM_RESERVE, // 提交并保留内存
        PAGE_READWRITE          // 初始保护属性：可读可写
    );

    if (pMemory == NULL) {
        std::cerr << "VirtualAlloc failed: " << GetLastError() << std::endl;
        return 1;
    }

    std::cout << "Memory allocated at: " << pMemory << std::endl;

    // 更改内存保护属性为可执行
    DWORD oldProtect;
    if (!VirtualProtect(
        pMemory,                // 内存区域的起始地址
        4096,                   // 内存区域的大小
        PAGE_EXECUTE_READWRITE, // 新的保护属性：可执行、可读、可写
        &oldProtect             // 保存旧的保护属性
    )) {
        std::cerr << "VirtualProtect failed: " << GetLastError() << std::endl;
        VirtualFree(pMemory, 0, MEM_RELEASE); // 释放内存
        return 1;
    }

    std::cout << "Memory protection changed to PAGE_EXECUTE_READWRITE" << std::endl;

    // 释放内存
    if (!VirtualFree(pMemory, 0, MEM_RELEASE)) {
        std::cerr << "VirtualFree failed: " << GetLastError() << std::endl;
        return 1;
    }

    std::cout << "Memory released" << std::endl;
    return 0;
}
```
#### 重要模块解释
#####  `if (pMemory == NULL)`语句
本质就是检查申请是否成功，不成功或者成功都输出相关信息以便调试
```c++
if (pMemory == NULL) {
    std::cerr << "VirtualAlloc failed: " << GetLastError() << std::endl;
    return 1;
}
```
###### 作用
- 检查 `VirtualAlloc` 函数是否成功申请内存。
- 如果 `VirtualAlloc` 失败，`pMemory` 会返回 `NULL`。
###### 代码逻辑
1. **`pMemory == NULL`**：
    - 判断 `pMemory` 是否为 `NULL`。如果为 `NULL`，表示内存申请失败。    
2. **`std::cerr`**：
    - 使用标准错误流输出错误信息。    
    - `GetLastError()` 返回最近一次系统调用的错误代码，帮助定位问题。  
3. **`return 1;`**：
    - 如果内存申请失败，程序返回错误代码 `1` 并退出。

##### `std::cout << "Memory allocated at: " << pMemory << std::endl;`语句
本质也是一种调试信息输出函数
```c++
std::cout << "Memory allocated at: " << pMemory << std::endl;
```
###### 作用
- 如果 `VirtualAlloc` 成功申请内存，输出分配的内存地址
###### 代码逻辑
1. **`std::cout`**：
    - 使用标准输出流打印信息。    
2. **`"Memory allocated at: "`**：
    - 输出提示信息，表示内存申请成功。   
3. **`pMemory`**：
    - 输出 `VirtualAlloc` 返回的内存地址（指针值）。  
4. **`std::endl`**：
    - 换行并刷新输出缓冲区。

##### 内存属性更改函数
```c++
DWORD oldProtect;
    if (!VirtualProtect(
        pMemory,                // 内存区域的起始地址
        4096,                   // 内存区域的大小
        PAGE_EXECUTE_READWRITE, // 新的保护属性：可执行、可读、可写
        &oldProtect             // 保存旧的保护属性
    )) {
        std::cerr << "VirtualProtect failed: " << GetLastError() << std::endl;
        VirtualFree(pMemory, 0, MEM_RELEASE); // 释放内存
        return 1;
    }
```
其中的`DWORD oldProtect;`语句就是c语言中的变量声明，用于定义一个名为 `oldProtect` 的变量，其类型为 `DWORD`。

其中`if`函数就是为了判断内存属性是否更改成功并且生成相对应的调试信息
```c++
if (!VirtualProtect(
        pMemory,                // 内存区域的起始地址
        4096,                   // 内存区域的大小
        PAGE_EXECUTE_READWRITE, // 新的保护属性：可执行、可读、可写
        &oldProtect             // 保存旧的保护属性
    )) 
```
###### 代码逻辑
1. **`if (!VirtualProtect(...))`**：
    - 调用 `VirtualProtect` 函数，并检查其返回值。    
    - 如果返回值为 `FALSE`，表示更改保护属性失败。   
2. **`std::cerr`**：
    - 使用标准错误流输出错误信息。   
    - `GetLastError()` 返回最近一次系统调用的错误代码，帮助定位问题。    
3. **`VirtualFree(pMemory, 0, MEM_RELEASE)`**：
    - 如果更改保护属性失败，使用 `VirtualFree` 释放之前申请的内存。
    - `MEM_RELEASE` 表示释放整个内存区域。    
4. **`return 1;`**：
    - 如果更改保护属性失败，程序返回错误代码 `1` 并退出。


## malloc函数
`malloc` 是 C 标准库中的一个函数，用于动态分配内存。它从堆（heap）中分配指定大小的内存块，并返回指向该内存块的指针。
### malloc函数解释
函数原型
```c++
void* malloc(size_t size);
```
#### 参数
- **`size`**：
    - 要分配的内存大小（以字节为单位）。  
    - 如果 `size` 为 0，`malloc` 的行为是未定义的（具体实现可能返回 `NULL` 或一个特殊的指针）。    
#### 返回值
- 如果成功，返回指向分配的内存块的指针。
- 如果失败，返回 `NULL`。
#### 特点
- **动态分配**：`malloc` 在运行时动态分配内存，适用于需要灵活管理内存的场景。
- **不初始化内存**：`malloc` 分配的内存块是未初始化的，内容可能是随机的。
- **手动释放**：使用 `malloc` 分配的内存必须手动释放，否则会导致内存泄漏。

### malloc函数使用事项
1. **调用 `malloc`**：
    - 指定需要分配的内存大小。    
    - 检查返回值是否为 `NULL`，确保内存分配成功。
2. **使用分配的内存**：
    - 将返回的指针强制转换为所需的类型（如 `int*`、`char*` 等）。
    - 对内存进行读写操作。    
3. **释放内存**：
    - 使用 `free` 函数释放分配的内存，避免内存泄漏。
<mark style="background: #FF5582A6;">重点注意返回的是指针以及该指针的类型</mark>

### malloc函数示例代码
```c++
#include <stdio.h>
#include <stdlib.h>

int main() {
    // 申请内存
    int* ptr = (int*)malloc(10 * sizeof(int)); // 分配 10 个 int 类型的内存空间
    if (ptr == NULL) {
        printf("Memory allocation failed!\n");
        return 1;
    }
    // 使用内存，示例代码通过for循环，实际代码可以使用其他方法
    for (int i = 0; i < 10; i++) {
        ptr[i] = i + 1; // 初始化内存
    }
    // 释放内存
    free(ptr);
    ptr = NULL; // 将指针置为 NULL，避免悬空指针

    printf("Memory freed.\n");
    return 0;
}
```

<mark style="background: #FF5582A6;">重点关注malloc函数的使用，以及内存的手动释放其他的不用过多关注</mark>


## HeapCreate函数
`HeapCreate` 是 Windows API 中的一个函数，用于创建一个新的私有堆（private heap）。私有堆是进程内存中独立管理的一块区域，允许程序通过 `HeapAlloc`、`HeapFree` 等函数动态分配和释放内存。
### HeapCreate函数详解
函数原型
```c++
HANDLE HeapCreate(
  DWORD  flOptions,     // 堆的选项（通常为 0）
  SIZE_T dwInitialSize, // 堆的初始大小（字节）
  SIZE_T dwMaximumSize  // 堆的最大大小（字节）
);
```
#### 参数说明
- **`flOptions`**：
    - 堆的创建选项，常用的值有：
        - **`0`**：默认选项，堆不可动态扩展（若指定 `dwMaximumSize` 为 0 则允许动态扩展）。
        - **`HEAP_NO_SERIALIZE`**：禁用多线程同步（需确保堆不会被多线程访问）。  
- **`dwInitialSize`**：
    - 堆的初始大小（字节）。如果为 0，系统会自动分配一个页的大小。
- **`dwMaximumSize`**：
    - 堆的最大大小（字节）。如果为 0，堆可以动态增长。   
#### 返回值
- 成功时返回堆的句柄（`HANDLE`）。
- 失败时返回 `NULL`。

### 示例代码
```c++
#include <windows.h>
#include <stdio.h>

int main() {
    // 1. 创建一个私有堆
    HANDLE hHeap = HeapCreate(
        0,              // 默认选项（堆可动态扩展）
        0,              // 初始大小（由系统决定）
        0               // 最大大小（允许动态扩展）
    );
    // 错误信息显示
    if (hHeap == NULL) {
        return 1;
    }

    // 2. 从堆中分配内存
    const SIZE_T bufferSize = 100; // 分配 100 字节
    LPVOID pBuffer = HeapAlloc(
        hHeap,          // 堆句柄
        HEAP_ZERO_MEMORY, // 初始化内存为 0
        bufferSize      // 分配的大小
    );

    if (pBuffer == NULL) {
        return 1;
    }

    // 4. 释放内存并销毁堆
    if (!HeapFree(hHeap, 0, pBuffer)) {
        printf("HeapFree failed! Error: %d\n", GetLastError());
    }

    if (!HeapDestroy(hHeap)) {
        printf("HeapDestroy failed! Error: %d\n", GetLastError());
    }

    return 0;
}
```

<mark style="background: #FF5582A6;">重点关注其中利用HeapCreate函数申请堆和利用HeapAlloc函数在堆中申请内存，以及最后释放内存销毁堆的函数以及利用，内存放入数据的过程不用在意。</mark>

注意事项：- **内存泄漏**：务必调用 `HeapFree` 和 `HeapDestroy` 释放资源。