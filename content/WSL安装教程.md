# 1. 建目录
mkdir D:\WSL

# 2. 下载（文件名完全一致）
Invoke-WebRequest -Uri "https://cloud-images.ubuntu.com/wsl/releases/24.04/current/ubuntu-noble-wsl-amd64-24.04lts.rootfs.tar.gz" -OutFile "E:\WSL\u24.tar.gz"

# 3. 导入到 D 盘
wsl --import Ubuntu-KDE E:\WSL\Ubuntu-KDE E:\WSL\u24.tar.gz --version 2

# 4. 查看结果
wsl -l -v

wsl su 密码：111
# 删除刚才放在 D 盘的实例目录
Remove-Item -Recurse -Force "D:\WSL"

# 可选：删除默认 Ubuntu（如果之前装过）
Remove-Item -Recurse -Force "$env:LOCALAPPDATA\Packages\CanonicalGroupLimited.UbuntuonWindows_*"


你已经下载好了 Ubuntu 的 `.tar.gz` 文件（如 `u24.tar.gz`），接下来可以通过 **WSL 命令行** 将其导入为 WSL2 发行版。以下是详细步骤：

---

### ✅ 步骤 1：启用 WSL 功能（如果尚未启用）

以管理员身份运行 PowerShell 或 CMD，执行以下命令：

```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

然后启用虚拟机平台（WSL2 需要）：

```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

> ⚠️ 执行完这两个命令后，**重启电脑**。

---

### ✅ 步骤 2：设置 WSL 版本为 2

打开 PowerShell（不需要管理员权限）：

```powershell
wsl --set-default-version 2
```

---

### ✅ 步骤 3：导入已下载的 `.tar.gz` 文件

#### 📌 重要说明：
你的文件是 `u24.tar.gz`，这通常是 Ubuntu 24.04 的压缩包。你需要使用 `wsl --import` 命令来导入它。

#### 🔧 操作步骤：

1. 打开 **PowerShell**（普通用户即可）
2. 使用 `wsl --import` 命令，格式如下：

```powershell
wsl --import Ubuntu E:\WSL\Ubuntu u24.tar.gz
```

> 💡 解释：
> - `Ubuntu`：这是你给这个发行版起的名字（可自定义）
> - `E:\WSL\Ubuntu`：这是该发行版在磁盘上的安装路径（建议创建一个文件夹）
> - `u24.tar.gz`：你的压缩包路径（必须是完整路径或相对路径）

✅ 示例（推荐写完整路径）：

```powershell
wsl --import Ubuntu "E:\WSL\Ubuntu" "E:\WSL\u24.tar.gz"
```

> ❗注意：路径中如果有空格，请用引号包围。

---

### ✅ 步骤 4：设置默认用户（可选）

导入后，首次运行会提示你设置用户名和密码。但如果你希望直接设置默认用户，可以这样做：

```powershell
wsl --user root -d Ubuntu
```

然后手动设置用户（例如添加新用户并设置密码）：

```bash
adduser yourusername
```

---

### ✅ 步骤 5：启动 Ubuntu

运行：

```powershell
wsl -d Ubuntu
```

或者直接输入：

```powershell
Ubuntu
```

> 如果你设置了默认发行版，直接输入 `wsl` 即可。

---

### ✅ 可选：设置默认发行版

```powershell
wsl --set-default Ubuntu
```

以后只需输入 `wsl` 就能启动 Ubuntu。

---

### 🛠️ 常见问题解决

| 问题 | 解决方法 |
|------|----------|
| `The specified distribution was not found` | 确保路径正确，且文件未损坏 |
| `Error: Cannot find the specified file` | 检查 `.tar.gz` 是否存在、是否被压缩错误 |
| `Failed to import` | 确保 WSL 已启用，且版本为 2 |

---

### ✅ 总结

你只需要执行：

```powershell
wsl --import Ubuntu "E:\WSL\Ubuntu" "E:\WSL\u24.tar.gz"
```

然后通过 `wsl -d Ubuntu` 启动即可！

---

如有需要，我也可以帮你写一个脚本自动完成这些操作。欢迎继续提问！


## 问题汇总
1. 如果wsl安装不了总是报错，先把所有的驱动更新完成
2. 然后记得打开windows的虚拟平台监控，windows的子系统，虚拟平台
3. 你可以先安装wsl应用程序后，再去import你原先的镜像压缩包，或者把原先的镜像包导出
4. 用户的创建需要先把wsl的开机系统全部关闭之后再去重新设置用户名

  

