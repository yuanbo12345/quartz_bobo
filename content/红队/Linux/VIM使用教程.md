# 配置路径
![[image-229.png|557x381]]
# 搜索相关配置修改
![[image-230.png]]

![[image-231.png]]

![[image-232.png]]

### 1. 查找相关的主要配置

#### （1）基本查找设置

在 `~/.vimrc`（Vim）或 `~/.config/nvim/init.vim`（Neovim）中添加以下内容：

vim
```bash
" 查找时高亮所有匹配项
set hlsearch

" 输入搜索内容时实时高亮（增量搜索）
set incsearch

" 忽略大小写（除非搜索词包含大写字母）
set ignorecase
set smartcase

" 搜索到文件末尾时循环回到开头（默认开启，如需关闭）
set nowrapscan  " 关闭循环搜索
```

### **（2）快捷键优化**

vim
```bash
" 按 <Esc> 取消高亮（替代默认的 :nohlsearch）
nnoremap <silent> <Esc> :nohlsearch<CR>

" 用 * 和 # 搜索时，不跳转到下一个匹配项（保持光标位置）
nnoremap * *N
nnoremap # #N
```


# Vim更新
```bash
sudo apt update
sudo apt upgrade vim
```
