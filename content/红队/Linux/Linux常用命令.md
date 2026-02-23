## 文件操作
### 上传文件
#### 使用scp命令，基于ssh协议
```bash
# 上传单个文件到远程主机的家目录
scp /path/to/local/file.txt username@vps_ip:/home/username/

# 上传整个目录（使用 -r 递归选项）
scp -r /path/to/local/directory username@vps_ip:/path/on/vps/

# 指定端口（如果SSH不是默认22端口）
scp -P 2222 file.txt username@vps_ip:/remote/path/
```

#### 使用rsync进行大文件上传
```bash
# 上传文件（保持权限和时间戳）
rsync -avz /local/path/file.txt username@vps_ip:/remote/path/

# 上传整个目录
rsync -avz /local/directory/ username@vps_ip:/remote/directory/

# 部分传输（断点续传）
rsync -avz --partial /large/file username@vps_ip:/remote/path/
```

### 隐藏文件夹或者文件
1. 在linux中隐藏文件夹或者文件通常都以`.`放到文件前边，例如`.config`
#### 显示隐藏文件或者文件夹
```bash
ls -a  # 显示所有文件，包括隐藏文件（.和..也会显示）
ls -A  # 显示所有文件，包括隐藏文件（但不显示.和..）
```

#### 创建或者将文件变成隐藏文件
```bash
mkdir config .config

mv config .config
```
