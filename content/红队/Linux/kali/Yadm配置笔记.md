# YADM完全指南

  

## 目录

- [1. 什么是YADM](#1-什么是yadm)

- [2. 安装YADM](#2-安装yadm)

- [3. 初始配置](#3-初始配置)

- [4. 基本操作](#4-基本操作)

- [5. 文件管理](#5-文件管理)

- [6. 仓库管理](#6-仓库管理)

- [7. 分支管理](#7-分支管理)

- [8. 跟踪查看](#8-跟踪查看)

- [9. SSH连接问题排查](#9-ssh连接问题排查)

- [10. 恢复配置到新系统](#10-恢复配置到新系统)

- [11. 高级功能](#11-高级功能)

  

## 1. 什么是YADM

  

YADM（Yet Another Dotfiles Manager）是一个用于管理配置文件（dotfiles）的工具，它使用Git作为后端，但提供了更友好的界面和专为配置文件设计的功能。

  

YADM允许您：

- 跟踪配置文件的变化

- 在多台计算机之间同步配置

- 保持配置文件的版本历史

- 为不同的计算机使用不同的配置

- 安全地存储敏感文件

  

## 2. 安装YADM

  

### Ubuntu/Debian

```bash

sudo apt update

sudo apt install yadm

```

  

### CentOS/RHEL

```bash

sudo yum install epel-release

sudo yum install yadm

```

  

### Arch Linux

```bash

sudo pacman -S yadm

```

  

### 通过Git安装（适用于任何Linux发行版）

```bash

git clone https://github.com/TheLocehiliosan/yadm.git ~/.yadm-project

sudo ln -sf ~/.yadm-project/yadm /usr/local/bin/yadm

```

  

### 验证安装

```bash

yadm --version

```

  

## 3. 初始配置

  

### 初始化YADM仓库

```bash

yadm init

```

**解释**：创建一个新的YADM仓库，类似于`git init`，但仓库存储在`~/.local/share/yadm/repo.git`。

  

### 配置Git用户信息

```bash

yadm config --global user.name "您的姓名"

yadm config --global user.email "您的邮箱"

```

**解释**：设置Git提交时使用的用户信息，这些信息会显示在提交历史中。

  

### 添加远程仓库

```bash

yadm remote add origin git@github.com:用户名/仓库名.git

```

**解释**：连接本地YADM仓库到GitHub远程仓库，用于同步配置。

  

## 4. 基本操作

  

### 添加文件

```bash

# 添加单个文件

yadm add ~/.bashrc

  

# 添加多个文件

yadm add ~/.vimrc ~/.tmux.conf

  

# 添加目录中的所有文件

yadm add ~/.config/

  

# 添加所有修改的文件（已跟踪的）

yadm add -u

```

**解释**：将文件添加到YADM的跟踪列表，准备提交。`-u`选项只添加已经被跟踪且有变化的文件。

  

### 提交更改

```bash

# 基本提交

yadm commit -m "提交信息"

  

# 添加并提交所有更改

yadm commit -a -m "提交信息"

```

**解释**：将添加的文件变更保存到仓库历史中。`-a`选项会自动添加所有已跟踪的文件变更，无需先执行`yadm add`。

  

### 推送到远程仓库

```bash

# 基本推送

yadm push

  

# 首次推送设置上游分支

yadm push -u origin main

  

# 强制推送（谨慎使用）

yadm push -f

```

**解释**：将本地提交发送到远程仓库。`-u origin main`设置默认的上游分支，之后可以直接使用`yadm push`。

  

### 从远程仓库拉取

```bash

# 拉取更改

yadm pull

  

# 拉取并指定远程和分支

yadm pull origin main

```

**解释**：从远程仓库获取最新更改并合并到本地。

  

## 5. 文件管理

  

### 查看文件状态

```bash

yadm status

```

**解释**：显示当前工作目录中的文件状态，包括哪些文件被修改、添加或删除。

  

### 查看文件差异

```bash

# 查看所有差异

yadm diff

  

# 查看特定文件的差异

yadm diff ~/.bashrc

  

# 查看已添加但未提交的差异

yadm diff --staged

```

**解释**：显示文件的变更内容，对比工作目录与仓库中的文件。

  

### 撤销文件更改

```bash

# 撤销单个文件的更改

yadm checkout -- ~/.bashrc

  

# 撤销所有未提交的更改

yadm reset --hard

```

**解释**：恢复文件到最后一次提交的状态。`--hard`选项会丢弃所有未提交的更改，谨慎使用。

  

### 忽略文件

```bash

# 添加到全局忽略列表

yadm gitignore add "*.log"

  

# 编辑忽略文件

yadm gitignore edit

```

**解释**：设置YADM忽略特定文件，这些文件不会被跟踪或同步。

  

## 6. 仓库管理

  

### 查看仓库信息

```bash

# 查看远程仓库

yadm remote -v

  

# 查看本地配置

yadm config --list

```

**解释**：显示仓库的远程地址和配置信息。

  

### 更改远程仓库

```bash

# 修改远程仓库URL

yadm remote set-url origin 新的URL

  

# 添加额外的远程仓库

yadm remote add backup git@github.com:用户名/备份仓库.git

```

**解释**：更改或添加远程仓库地址，可以用于更换托管服务或添加备份。

  

### 查看仓库历史

```bash

# 查看提交历史

yadm log

  

# 查看简洁历史

yadm log --oneline

  

# 查看特定文件的历史

yadm log -p ~/.bashrc

```

**解释**：显示仓库的提交历史，`-p`选项可以显示每次提交的详细更改。

  

## 7. 分支管理

  

### 创建分支

```bash

# 创建新分支

yadm branch 分支名

  

# 创建并切换到新分支

yadm checkout -b 分支名

```

**解释**：创建新的分支用于开发新功能或测试配置，不影响主分支。

  

### 查看分支

```bash

# 查看本地分支

yadm branch

  

# 查看所有分支（包括远程）

yadm branch -a

  

# 查看详细信息

yadm branch -v

```

**解释**：显示仓库中的分支列表，当前分支会标记星号。

  

### 切换分支

```bash

yadm checkout 分支名

```

**解释**：切换到指定的分支，工作目录会更新为该分支的文件状态。

  

### 合并分支

```bash

# 合并指定分支到当前分支

yadm merge 分支名

  

# 合并并生成合并提交

yadm merge --no-ff 分支名

```

**解释**：将指定分支的更改合并到当前分支，`--no-ff`会生成一个专门的合并提交。

  

### 删除分支

```bash

# 删除已合并的分支

yadm branch -d 分支名

  

# 强制删除分支

yadm branch -D 分支名

```

**解释**：删除不再需要的分支，`-D`选项可以删除未合并的分支。

  

## 8. 跟踪查看

  

### 列出跟踪的文件

```bash

# 列出所有跟踪的文件

yadm list -a

  

# 以详细格式显示

yadm list -a -l

```

**解释**：显示YADM当前跟踪的所有文件列表。

  

### 查找未跟踪的文件

```bash

# 列出未跟踪的文件

yadm status -u

```

**解释**：显示工作目录中未被YADM跟踪的文件。

  

### 查看单个文件跟踪状态

```bash

yadm status ~/.某个文件

```

**解释**：查看特定文件的跟踪状态。

  

## 9. SSH连接问题排查

  

SSH连接是使用YADM远程同步的关键。以下是常见问题及解决方法：

  

### 生成SSH密钥

```bash

ssh-keygen -t ed25519 -C "your_email@example.com"

```

**解释**：创建一个新的SSH密钥对，用于GitHub身份验证。

  

### 添加SSH密钥到SSH代理

```bash

eval "$(ssh-agent -s)"

ssh-add ~/.ssh/id_ed25519

```

**解释**：启动SSH代理并添加私钥，避免每次都需要输入密码。

  

### 测试SSH连接

```bash

ssh -T git@github.com

```

**解释**：测试到GitHub的SSH连接是否正常，成功会显示欢迎消息。

  

### 诊断SSH问题

```bash

# 使用详细输出

ssh -vvv git@github.com

```

**解释**：显示详细的连接过程，有助于找出问题所在。

  

### 检查SSH配置

```bash

cat ~/.ssh/config

```

**解释**：查看SSH客户端配置，确保正确设置了GitHub的连接参数。

  

### 常见SSH错误及解决方案

  

#### 1. "Permission denied (publickey)"

- **原因**：GitHub上没有添加您的公钥或私钥权限不正确

- **解决**：

  ```bash

  # 显示公钥

  cat ~/.ssh/id_ed25519.pub

  # 复制并添加到GitHub设置

  # 修复权限

  chmod 600 ~/.ssh/id_ed25519

  chmod 644 ~/.ssh/id_ed25519.pub

  ```

  

#### 2. "Connection closed by remote host"

- **原因**：网络问题或SSH配置不正确

- **解决**：

  ```bash

  # 检查网络连接

  ping github.com

  # 使用配置文件

  echo "Host github.com\n  User git\n  IdentityFile ~/.ssh/id_ed25519" > ~/.ssh/config

  ```

  

#### 3. "Host key verification failed"

- **原因**：远程主机密钥变更

- **解决**：

  ```bash

  ssh-keygen -R github.com

  ```

  

## 10. 恢复配置到新系统

  

### 克隆仓库到新系统

```bash

# 使用SSH

yadm clone git@github.com:用户名/仓库名.git

  

# 使用HTTPS

yadm clone https://github.com/用户名/仓库名.git

```

**解释**：将您的配置仓库克隆到新系统，自动设置好YADM。

  

### 解密文件（如果有加密配置）

```bash

yadm decrypt

```

**解释**：解密使用YADM加密的敏感文件，如SSH密钥或API令牌。

  

### 应用模板（如果使用了模板）

```bash

yadm template

```

**解释**：应用特定于当前系统的模板文件。

  

### 安装依赖软件

如果您将软件包列表存储在仓库中：

```bash

# Debian/Ubuntu

xargs -a ~/.config/package-list.txt sudo apt install -y

```

**解释**：从保存的列表中安装所有必要的软件包。

  

### 恢复后检查状态

```bash

yadm status

```

**解释**：确认所有文件都已正确还原。

  

## 11. 高级功能

  

### 加密敏感文件

```bash

# 设置加密文件列表

mkdir -p ~/.config/yadm/encrypt

echo ".ssh/id_ed25519" > ~/.config/yadm/encrypt

  

# 加密文件

yadm encrypt

  

# 添加加密文件到仓库

yadm add ~/.local/share/yadm/archive

yadm add ~/.config/yadm/encrypt

```

**解释**：安全地存储敏感文件，如密钥或API令牌，避免明文保存在仓库中。

  

### 使用模板

```bash

# 创建模板

mkdir -p ~/.config/yadm/templates

cat > ~/.config/yadm/templates/bashrc.tmpl << 'EOF'

# 主机特定配置

{% if yadm.hostname == "laptop" %}

export PATH=$PATH:~/laptop-bin

{% else %}

export PATH=$PATH:~/server-bin

{% endif %}

EOF

  

# 应用模板

yadm template

```

**解释**：为不同的主机创建不同的配置文件，自动应用适合当前系统的设置。

  

### 自动化备份

```bash

# 创建备份脚本

cat > ~/bin/backup-config.sh << 'EOF'

#!/bin/bash

DATE=$(date +"%Y-%m-%d %H:%M:%S")

yadm add -u

if yadm status --porcelain | grep -q .; then

  yadm commit -m "自动备份: $DATE"

  yadm push

  echo "配置已备份"

else

  echo "无需备份"

fi

EOF

chmod +x ~/bin/backup-config.sh

  

# 设置定时任务

(crontab -l 2>/dev/null; echo "0 20 * * * ~/bin/backup-config.sh") | crontab -

```

**解释**：自动定期备份配置文件的变更，保持仓库更新。

  

### 别名设置

添加到`~/.bashrc`：

```bash

# YADM快捷命令

alias ys='yadm status'

alias ya='yadm add'

alias yc='yadm commit -m'

alias yp='yadm push'

alias yl='yadm pull'

```

**解释**：创建简短的命令别名，简化日常使用。

  

## 总结

  

YADM是一个强大的配置管理工具，它将Git的版本控制能力与专为配置文件设计的功能相结合。通过本指南中的命令，您可以有效地管理、同步和恢复您的配置文件，确保在任何系统上都能获得一致的工作环境。

  

记住，配置管理是一个持续的过程，随着您的工作习惯变化，定期更新和整理您的配置文件是保持高效工作的关键。