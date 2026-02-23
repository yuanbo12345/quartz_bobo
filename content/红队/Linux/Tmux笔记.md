# 基本概念区分
1. 区分好窗口和会话，会话是窗口的上一级
## 会话相关命令
```bash
// 创建新会话
tmux new -s niubibi
// 连接某个会话
tmux attach -t niubibi
// 删除某个会话
tmux kill-session -t niubibi
// 列出所有会话
tmux ls
```

> [!NOTE] 注意
> 1. 注意重新连接ssh之后，不能先使用tmux的快捷键，只有先连接上一个会话之后才可以
> 2. 当你已经在一个会话里时不能使用连接会话转到另一个会话中
> 3. 删除会话可以在另一个会话中直接删除另一个会话

## 窗口相关
1. 通过命令运行的基本上都是会话，窗口都是快捷键运行
```bash
Ctrl+b c         # 创建新窗口
Ctrl+b n         # 切换到下一个窗口
Ctrl+b p         # 切换到上一个窗口
Ctrl+b 0-9       # 切换到指定编号窗口
Ctrl+b w         # 显示窗口列表
Ctrl+b d         # 分离会话（保持后台运行）
Ctrl+b %         # 垂直分割窗格
Ctrl+b "         # 水平分割窗格
Ctrl+b 方向键     # 在窗格间切换

```

1. 删除某个窗口就是前缀+w先打开窗口列表，然后选中按下x按键，输入y确认即可

## 实现ssh断链保持历史记录
1. 创建配置文件
```bash
# 创建配置文件 nano ~/.tmux.conf
```

在配置文件中添加
```bash
# 保持会话持久化
set -g default-terminal "screen-256color"
set -g history-limit 10000
set -g mouse on

# 自动保存和恢复会话
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-resurrect'
set -g @plugin 'tmux-plugins/tmux-continuum'
set -g @continuum-restore 'on'
set -g @continuum-save-interval '15'
```

1. 日常使用
```bash
# 在tmux会话中正常工作
# 当需要断开Tabby时，按 Ctrl+b d 分离会话
# 或者直接关闭Tabby，tmux会话继续在后台运行
# 重新ssh之后，一定通过连接会话命令先连接再继续其他操作
```