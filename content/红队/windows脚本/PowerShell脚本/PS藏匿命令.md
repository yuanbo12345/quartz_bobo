### 隐藏 PowerShell 窗口
默认情况下，PowerShell 执行时会打开一个窗口，用户可能会注意到。可以使用以下参数隐藏窗口：
```powershell
powershell -WindowStyle Hidden -NoProfile -NonInteractive -ExecutionPolicy Bypass -File script.ps1
```

- `-WindowStyle Hidden`：隐藏 PowerShell 窗口
- `-NoProfile`：避免加载用户 PowerShell 配置，提高执行速度
- `-NonInteractive`：不显示交互式输入
- `-ExecutionPolicy Bypass`：绕过执行策略（==十分重要==）
这个方法适用于 **直接执行的 PowerShell**，但进程仍然可见。

所有需要调用外部进程或者创建新进程或者会话时，或者保存文件的命令都会出现窗口，都需要藏匿窗口。
