# NodePass-DNS
1. 容器安装，docker命令
## 安装教程
docker命令
```bash
docker run -d --name np-dns --restart=always \
  -p 53:53/udp \
  -e DOMAIN=lan \
  -e TARGET_IP=192.168.1.88 \
  yosebyte/nodepass
```