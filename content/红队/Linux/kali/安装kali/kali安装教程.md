### 虚拟机安装
教程网址--https://blog.csdn.net/SHERLOCK0226/article/details/139749187

### 配置说明
#### root用户设置密码
```bash
sudo passwd root
```

#### 配置软件源
1---编辑软件源
```bash
sudo vim /etc/apt/sources.list
```

2---替换软件源
```
deb http://mirrors.tuna.tsinghua.edu.cn/kali kali-rolling main non-free contrib
deb-src http://mirrors.tuna.tsinghua.edu.cn/kali kali-rolling main non-free contrib
```

```
deb http://mirrors.ustc.edu.cn/kali kali-rolling main non-free contrib
deb-src http://mirrors.ustc.edu.cn/kali kali-rolling main non-free contrib
```

```
deb http://mirrors.aliyun.com/kali kali-rolling main non-free contrib
deb-src http://mirrors.aliyun.com/kali kali-rolling main non-free contrib
```

3---更新软件包索引
```bash
sudo apt update
```

4---升级系统中的软件包
```bash
sudo apt full-upgrade -y
```

#### 配置中文输入法
1---更新软件源

2---安装fcitx框架以及Google输入法
```bash
sudo apt-get update
sudo apt-get install fcitx fcitx-googlepinyin
```

3---重启
```bash
sudo reboot
```


#### 查看电脑网络情况
##### 查看本机ip
```bash
ifconfig

ip addr show
```

##### 查看开放端口
```bash
netstat -tuln
```

使用 `lsof` 命令：
```bash
lsof -i :80
```

查看特定端口

##### 查看端口对应的服务
```bash
netstat -tulnp

ss -tulnp
```

参数 `-p` 会显示进程的 PID 和名称

#### 配置ssh服务
##### 配置ssh服务duan

作为ssh的服务端只需要安装，配置，开启ssh服务即可，客户端连接的时候使用的用户名密码就是服务端系统本身的账户名和密码，第一次连接后会自动生成接受密钥（公有私有密钥模式）

1--安装ssh服务
```bash
sudo apt install openssh-server
```

2--配置ssh服务
```bash
vim /etc/ssh/sshd_config
```

1. - 禁用 root 用户登录**（推荐）： 找到以下行：
        
        `PermitRootLogin yes`
        
        修改为：
        
        `PermitRootLogin no`
        
        这样可以提高安全性，避免直接通过 root 用户登录。
        
    - 更改默认端口**（可选）： 默认 SSH 端口为 22，你可以更改端口以提高安全性。例如，将端口改为 2222：
        
        `Port 2222`
        
        修改后，需要在防火墙中允许新端口（见步骤 4）。
        
    - 启用公钥认证**（推荐）： 找到以下行：
        
        `PubkeyAuthentication yes`
        
        确保它没有被注释掉。
        
    - 禁用密码认证**（可选，但推荐）： 找到以下行：
        
        `PasswordAuthentication yes`
        
        修改为：
        
        `PasswordAuthentication no`
        
        这样可以强制使用密钥对登录，提高安全性。

3--启动ssh服务
```bash
sudo systemctl start ssh
```
设置ssh服务自启
```bash
sudo systemctl enable ssh
```
检查服务状态
```bash
sudo systemctl status ssh
```

4--配置防火墙（放开端口）
允许ssh默认端口
```bash
sudo ufw allow ssh
```
允许自定义端口
```bash
sudo ufw allow 2222/tcp
```

###### 配置ssh客户端
1--连接服务端
```bash
ssh username@remote_host_ip
```
使用了其他端口
```bash
ssh -p 2222 username@remote_host_ip
```
会提示输入密码即可、

<mark style="background: #FF5582A6;">ssh服务可以在服务端-客户端之间相互通信，也可以在客户端之间相互通信</mark>
