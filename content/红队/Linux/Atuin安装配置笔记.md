# 使用apt安装
```bash
# 添加 Atuin 的官方仓库 最好使用clash——linux项目代开tun后下载
curl https://raw.githubusercontent.com/atuinsh/atuin/main/install.sh | bash

# 更新软件包列表
sudo apt update

```

## Bash集成

1. 打开编辑 
```bash
vi ~/.bashrc

// 添加一下命令
# 让 atuin 不抢任何键，只保留 Ctrl-R
export ATUIN_NOBIND=true
eval "$(atuin init bash)"
bind -x '"\C-r": __atuin_history'

// 保存后执行下面脚本
source ~/.bashrc
## atuin导入历史记录
```
导入历史记录

```bash
atuin import auto
```

## 设置atuin回车命令放置
先不执行
1. 先打开atuin的路径
```bash
vi ~/.config/atuin/config.toml
```

2. 找到 enter_accept = false
```bash
enter_accept = false
```

## 快捷键绑定禁用
```bash
在bash中添加--disable-up-arrow 
vi ~/.bashrc

eval "$(atuin init zsh --disable-up-arrow)"

完成后重开终端
```