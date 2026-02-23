# 安装条件
1. 安装好docker，配置好国内景象源
2. 安装好docker compose插件
```bash
sudo apt-get update
sudo apt-get install docker-compose-plugin

// 验证
docker compose version
```

## 安装项目
1. 常见自己要安装的目录
2. 然后按照booklore官方文档安装方法，一定要逐步安装，先是目录，再是配置文件，修改配置文件的目录为本地目录
3. 官方教程文档网址：https://adityachandelgit.github.io/booklore-docs/docs/getting-started


> [!NOTE] 注意
> 1. 想要添加库，要现在bookdrop这个目录下添加一本书，让项目识别一下
> 2. 电子书网站：https://github.com/TapXWorld/ChinaTextbook

## Docker常用命令关于这个项目
1. 卸载安装的容器
```bash
// 卸载名字为mariadb的容器，这是这个项目出错，卸载这个
docker rm -f mariadb booklore
```

## compose配置文件一定要改对
<mark style="background: #FF5582A6;">路径一定不能错</mark>
示例
```yml
services:
  booklore:
    image: ghcr.io/adityachandelgit/booklore-app:latest
    container_name: booklore
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - DATABASE_URL=jdbc:mariadb://mariadb:3306/booklore   # Only modify this if you're familiar with JDBC and your database setup
      - DATABASE_USERNAME=booklore                          # Must match MYSQL_USER defined in the mariadb container
      - DATABASE_PASSWORD=your_secure_password              # Use a strong password; must match MYSQL_PASSWORD defined in the mariadb container 
      - SWAGGER_ENABLED=false                               # Enable or disable Swagger UI (API docs). Set to 'true' to allow access; 'false' to block access (recommended for production).
    depends_on:
      mariadb:
        condition: service_healthy
    ports:
      - "6060:6060"
    volumes:
      - /home/yuanbo/桌面/Websit/booklore/data:/app/data       # Internal app data (settings, metadata, cache)
      - /home/yuanbo/桌面/Websit/booklore/books:/books1       # Book library folder — point to one of your collections
      - /home/yuanbo/桌面/Websit/booklore/books:/books2       # Another book library — you can mount multiple library folders this way
      - /home/yuanbo/桌面/Websit/booklore/bookdrop:/bookdrop   # Bookdrop folder — drop new files here for automatic import into libraries
    restart: unless-stopped

  mariadb:
    image: lscr.io/linuxserver/mariadb:11.4.5
    container_name: mariadb
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - MYSQL_ROOT_PASSWORD=super_secure_password  # Use a strong password for the database's root user, should be different from MYSQL_PASSWORD
      - MYSQL_DATABASE=booklore
      - MYSQL_USER=booklore                        # Must match DATABASE_USERNAME defined in the booklore container
      - MYSQL_PASSWORD=your_secure_password        # Use a strong password; must match DATABASE_PASSWORD defined in the booklore container
    volumes:
      - /home/yuanbo/桌面/Websit/booklore/config/mariadb:/config
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "mariadb-admin", "ping", "-h", "localhost"]
      interval: 5s
      timeout: 5s
      retries: 10

```