## 随记
1. 后渗透阶段和edr的主要对抗就是：规避行为检测/内存检测
2. 如果采用工具的方式进行后渗透，可以利用工具将pe文件转换成shellcode之后，通过loader上传到目标主机之后再去加载，加载loader时可以利用CS的`execute-assembly` 或 `inline-execute`
3. 不采用工具进行后渗透可以采用CS中的BOF进行后渗透，那问题就是要验证bof的免杀时效性
4. IAT Hook是EDR将PE文件解析之后的IAT表中的所有函数地址进行了patch到杀软自写的jmp地址中以检测调用api的参数，顺序之类的东西。绕过的办法就是动态加载api或者间接调用syscall


## 规避行为检测
1. 到现在为止规避行为检测最有效的方式只有三种：直接系统调用，间接系统调用，行走UnHook NTDLL（修复内存中的Hook）
### UnHook NTDLL
#### 技术原理：
- EDR Hook 的是内存中的 `ntdll.dll`
- 我们可以从磁盘重新加载干净的 ntdll.dll，然后**覆盖原来的钩子部分**
- 手动 Patch 掉那些 `jmp` 指令，把函数还原成原版    
#### 实现方式：
- 加载磁盘文件 `ntdll.dll` 的副本（用 `LoadLibraryEx` 加到内存中）
- 用副本的原始字节覆盖当前进程中的 ntdll.dll 代码段（text段）
- 或者直接复制整个 `.text` 节区（更粗暴）
#### 工具：
- [Unhooking NTDLL with memcpy](https://github.com/Cracked5pider/NoHook)
- [nina](https://github.com/Ch0pin/Hotfix): Hook fixing + patchguard bypass
- Cobalt Strike BOF: `bypass-edr-memory-patching.bof`
#### 缺点：
- 某些EDR会检测 `.text` 段是否被修改（memory integrity）    
- 如果操作不当，可能造成程序崩溃（但一般不会）


## 后渗透阶段
### 信息收集
1. 木马免杀上线之后，首先要做到的就是信息收集
2. 信息收集的要点：不要使用敏感的命令，不使用敏感的进程链，最好用CS的BOF去做
常见的CS的信息收集BOF如下所示
|工具|用途|
|---|---|
|`whoami.bof`|当前用户名、权限、会话状态|
|`getprivs.bof`|查看当前 Token 拥有哪些特权（如 SeDebugPrivilege）|
|`rev2self` / `steal_token`|会话令牌操作（域环境常见）|
|`Seatbelt.bof`|收集系统配置、计划任务、UAC设置|
|`SharpUp.bof`|提权分析与建议|
|`Find-AVSignature.bof`|探测 EDR/杀毒软件存在与签名模式|

这些bof都是使用beacon的进程在内存中直接运行相关的API以完成操作的，所以自己BOF就要注意规避API HOOK和API 进程链检测以及内存dump或者内存扫描的检查

### 权限提升
1. 现代红队的权限提升采用的实用性方法不多
2. 现在最全面精确的规避方式就是使用bof，但是最好保证bof实现相关功能的底层原理是调用syscall的api

#### 采用令牌盗取（伪装）
- **原理**：当前系统中存在 SYSTEM 权限的会话（如服务），可复制其 Token 使用
- **实现方式**：使用 `steal_token`, `make_token`, `rev2self` 等 BOF
- **绕过点**：不产生进程创建、无暴力调用，极隐蔽

#### UAC Bypass（绕过用户账户控制）

- **原理**：Windows 某些程序被信任可提权启动，可被劫持
- **常用方式**：
    - **fodhelper**、**eventvwr**、**sdclt** 劫持注册表    
- **实现工具**：`UACMe`、`bof-uac-bypass`（需改代码）
- **绕过点**：
    - 无新进程或通过合法系统程序启动    
    - 可以用 BOF 实现注册表写入，绕开命令行检测

#### 服务提权（Service misconfig）

- **原理**：某些服务可被低权限用户修改路径或配置
- **方法**：
    - `sc qc <服务名>` 查看权限    
    - 用 BOF 修改 `ImagePath` 为你控制的执行文件    
- **绕过点**：
    - 操作合法服务、无新建文件落地

重点注意：实战中避免 Drop 外部二进制工具（如mimikatz.exe），改为内存加载（如mimilib.dll）

### 权限维持
#### 注册表 Run Key（常用）
- 写入 `HKCU\Software\Microsoft\Windows\CurrentVersion\Run    
- 启动时运行你的 Beacon Loader
- **绕过点**：
    - BOF 写入注册表，无落地脚本    
    - 可配置成内存shellcode loader
    - 
#### 计划任务（Scheduled Task）
- 创建隐藏计划任务，定时或开机启动
- `schtasks /create ...`
- **绕过点**：
    - 用BOF或COM接口创建任务，避免命令行关键词命中

#### 服务注册（Service Persistence）
- 创建一个后台服务并设置为开机启动
- 可使用合法服务名伪装
- **绕过点**：
    - 使用 `CreateServiceA` BOF 执行，行为非常干净

#### WMI Event Subscription（高级）
- 创建WMI事件，在触发时启动 loader
- 极其隐蔽，EDR很难追踪
- 工具：`wmi-persistence.bof`

#### LSASS 注入（注意风险）
- 注入Shellcode进lsass.exe等长驻进程
- 难度高，被抓住代价大
- 通常用于高隐蔽渗透中，配合 indirect syscall