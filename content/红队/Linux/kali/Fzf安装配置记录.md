其实atuin和这个fzf的功能基本一样，有atuin就可以了，但是fzf支持显示路径和相关东西

## 安装过程
1. 项目下载最新的release版本
2. 解压
```bash
tar -xzf fzf-0.64.0-linux_amd64.tar.gz
```
3. 给解压后的文件权限执行，解压后是二进制文件
```bash
chmod +x fzf
// 执行的方法二
sudo mv fzf /usr/local/bin/
```

## 快捷键修改
1. 原生绑定的快捷键![[image-243.png]]

# windows安装
这是一份关于 `fzf` 和 `PSFzf` 在 Windows PowerShell 中安装和配置的完整指南，以及您遇到的常见问题及解决方案汇总。

您可以直接复制并在您的文档中使用。

---

# Fzf 和 PSFzf 在 Windows PowerShell 中的配置指南

本文档旨在提供在 Windows PowerShell 环境中安装和配置 `fzf` 和 `PSFzf` 模块的完整步骤，并总结了配置过程中可能遇到的常见问题及其解决方法。

## 一、环境准备与模块安装

### 1. 安装 Fzf 二进制文件 (`fzf.exe`)

`PSFzf` 模块需要底层工具 `fzf.exe` 才能工作。

- **方法 A (推荐，使用包管理器)**：
    
    PowerShell
    
    ```
    # 需要安装 Scoop 或 Chocolatey
    scoop install fzf 
    # 或者
    choco install fzf
    ```
    
- **方法 B (手动安装)**：
    
    1. 从 [fzf 的 GitHub 仓库](https://github.com/junegunn/fzf/releases) 下载最新版本的 `fzf-*-windows_amd64.zip`。
        
    2. 解压，并将 `fzf.exe` 文件放置到一个已添加到**环境变量 `Path`** 的目录中（例如 `C:\Windows\System32` 或您的用户目录下的一个自定义 bin 文件夹）。
        
    3. 在 PowerShell 中输入 `fzf`，如果能运行则表示安装成功。
        

### 2. 安装 PSFzf 模块

在 **普通模式**（非管理员）的 PowerShell 窗口中运行以下命令，将模块安装到当前用户的作用域下：

PowerShell

```
# 1. 临时更改执行策略，允许运行脚本 (如果受限)
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser -Force

# 2. 安装 PSFzf 模块
Install-Module PSFzf -Scope CurrentUser -Force
```

## 二、PowerShell 配置文件配置

为了让 `PSFzf` 在每次启动时自动加载并正确绑定快捷键，您需要编辑您的 PowerShell 配置文件。

1. 打开配置文件：
    
    PowerShell
    
    ```
    notepad $PROFILE
    ```
    
2. **清空现有内容**（如果存在），然后**复制粘贴以下代码**。这段代码是根据您遇到的版本问题 (`PSFzf 2.7.3`) 特别修正的，使用了正确的内部函数名：
    

PowerShell

```
# 强制加载 PSFzf 模块
Import-Module PSFzf -Force

# 启用 PSFzf 的命令别名，例如 Invoke-Fzf
Enable-PsFzfAliases

# 核心修正：使用模块内正确的 PSReadLine 绑定函数名

# 绑定 Ctrl+R 到历史记录搜索
Set-PSReadLineKeyHandler -Chord 'Ctrl+r' -ScriptBlock { Invoke-FzfPsReadlineHandlerHistory }

# 绑定 Ctrl+T 到文件/目录搜索
Set-PSReadLineKeyHandler -Chord 'Ctrl+t' -ScriptBlock { Invoke-FzfPsReadlineHandlerProvider }

# 启动成功的提示（可根据需要修改或删除）
Write-Host "PSFzf 模块已成功加载并绑定快捷键！" -ForegroundColor Green
```

3. 保存文件并关闭记事本。
    
4. **重启**所有 PowerShell 窗口使配置生效。
    

## 三、常见问题与解决方法汇总

|**遇到问题**|**截图错误提示**|**原因分析**|**解决方法**|
|---|---|---|---|
|**Ctrl+R 无反应，但在管理员模式下可以**|-|**历史文件权限问题**：PowerShell 历史记录文件被创建为“仅管理员可写”。|删除或重命名历史文件：`Remove-Item (Get-PSReadLineOption).HistorySavePath -Force`。重启 PowerShell 会自动创建新文件。|
|**运行 `Set-PSReadLineKeyHandler -Chord ...` 报错**|`找不到与参数名称“Chord”匹配的参数`|**PSReadLine 版本过旧**：您的 PowerShell 版本太旧，不支持 `psfzf` 所需的新参数。|以**管理员身份**运行：`Install-Module PSReadLine -Force -AllowPrerelease -Scope AllUsers`，强制更新到最新版本。|
|**按下 `Ctrl+R` 后，提示“无法将‘Invoke-FzfHistory’识别为 cmdlet”**|`无法将‘Invoke-FzfHistory’识别为 cmdlet...`|**模块未加载或函数名错误**：要么 `Import-Module PSFzf` 失败，要么您使用的函数名不存在。|确保配置文件中有 `Import-Module PSFzf -Force`；**更重要的是，请使用本文档中提供的修正后的函数名**：`Invoke-FzfPsReadlineHandlerHistory`。|
|**模块加载成功（绿字），但按下快捷键仍报错**|`An exception occurred in custom key handler...`|**版本兼容性问题**：您的 `PSFzf` 版本（如 2.7.3）将快捷键函数名进行了更改。|**这是您最终的解决方案**：修改配置文件，使用该版本中实际存在的函数名：`Invoke-FzfPsReadlineHandlerHistory` 和 `Invoke-FzfPsReadlineHandlerProvider`。|
|**中文提示显示乱码**|`PSFzf 閺備澆鍟婂﹢`|**编码问题**：PowerShell 5.1 默认使用 GBK，而您的配置文件保存为 UTF-8。|**方法一**：将配置文件另存为时，选择 **`ANSI`** 编码。**方法二**：将配置文件的提示文本改为英文。|