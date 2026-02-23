## pe文件加载dll的过程
1. dll的导出函数代码在运行时会被映射到pe文件进程的内存地址中![[image-69.png]]
2. EDR的hook是怎么将dll注入的
3. ![[image-70.png]]

## ETW事件
#### ETW记录
1. 以一个pe文件加载之后修改注册表为例，看ETW整体的记录过程
2. 当pe文件被加载执行时会调用一系列的各种windows api函数执行，而这些函数会进行syscall的操作再去调用内核态的对应api，内核态中的ETW写入api函数会被触发记录下相关的参数。然后回到用户态层，也会调用用户态的ETWWrite写入函数写入用户态的ETW Buffer数据结构中，然后其中的内容会同步给早已经获取了ETW Session number的进程中
3. 流程图
```
 evil.exe ─→ RegSetValueExA 
      │
      ▼
 NtSetValueKey (Syscall)
      │
      ▼
  CmSetValueKey (内核)
      │
      ├─→ EtwWrite → EtwpEventWriteFull
      │             │
      │             └→ 写入 ETW Buffer
      │
      ▼
  ETW Buffer 满 / 定时
      │
      ▼
  EDR Session 回调
      │
      ▼
  EDR Consumer 进程 (Defender / Sysmon)
      │
      ▼
  分析+决策+响应

```

#### ETW决策响应
1. ETW函数再将所有buffer中的记录回调给各个订阅的软件之后，各个软件会根据buffer提供的各种个参数去个自己设立的规则对比，如果成功就会进行响应![[image-71.png]]
2. ![[image-72.png]]

## 内核态ETW和用户态ETW
1. 内核态事件记录和用户态事件记录是两个独立的记录系统，不过公用buffer中心和session guid号
2. 软件的api调用从内核态触发了一些敏感的内核态api，那么会直接进行内核态的etw事件写入，如果pe文件的调用先是触发了用户层的敏感函数api那么先有用户层的etw函数写入
3. 一个EDR会同时监控内核态和用户态的事件记录返回信息，同时和规则进行判断
#### 用户态ETW函数
1. ![[image-73.png]]

#### 内核态ETW函数
1. ![[image-74.png]]
2. ![[image-75.png]]
```
内核态行为（进程创建、注册表改动）
  │
  └─→ EtwWrite (内核) ──→ 内核 Session Buffer
                        │
                        └─→ Session Consumer (EDR)

用户态应用上报事件
  │
  └─→ EventWrite (用户)
        │
        └─→ NtTraceEvent (syscall)
              │
              └─→ EtwpEventWriteFull (内核)
                      │
                      └─→ 用户态 Session Buffer
                                │
                                └─→ Session Consumer (EDR)

```