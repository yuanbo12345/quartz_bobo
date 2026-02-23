# Linux安装
## Ubuntu环境
### 快速安装命令
```bash
curl -fsSL https://get.docker.com | sudo bash -s docker --mirror Aliyun
sudo usermod -aG docker $USER && newgrp docker
```
1. 自动识别 Ubuntu / CentOS / Debian 等发行版，卸载旧版、装依赖、配置官方仓库并安装最新稳定版 Docker（Engine + CLI + Compose 插件）。
2. 把当前用户加入 docker 组，立刻生效，无需再  sudo 。

### 配置国内镜像源环境
#### 方法一：命令配置
1. 创建配置文件
```bash
sudo mkdir -p /etc/docker

sudo tee /etc/docker/daemon.json >/dev/null <<'EOF'
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://dockerproxy.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://hub-mirror.c.163.com"
  ]
}
EOF

```

2. 重启服务，配置生效
```bash
sudo systemctl daemon-reload
sudo systemctl restart docker

```

3. 验证
```bash
docker info --format '{{json .RegistryConfig.Mirrors}}'
// 输出显示上面的镜像链接为配置正确
```

