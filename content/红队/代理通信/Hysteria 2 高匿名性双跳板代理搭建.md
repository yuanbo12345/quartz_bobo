# Hysteria 2 高匿名性双跳板代理搭建指南

  

本文档将指导您从零开始搭建一个基于 Hysteria 2 的高匿名、多链路代理服务。我们将采用“客户端 -> 服务器A (入口) -> 服务器B (出口) -> 目标网站”的架构，逐步实现以下功能：

  

1.  **基础双跳板代理**：实现基本的流量转发链。

2.  **流量混淆与安全增强**：为代理流量添加加密混淆，提升安全性。

3.  **域前置 (Domain Fronting)**：利用 CDN 隐藏真实服务器 IP，增强抗封锁能力。

4.  **反向代理 (可选)**：让代理服务与网站在同一端口共存，提升隐蔽性。

  

---

  

## 整体架构


我们的目标架构如下：
 

```

┌──────────┐   (加密连接)   ┌─────────────┐   (加密连接)   ┌─────────────┐   (原始流量)   ┌──────────┐

│          │                │             │                │             │                │          │

│  客户端  ├───────────────►│  服务器 A   ├───────────────►│  服务器 B   ├───────────────►│ 互联网   │

│ (Your PC)│                │ (入口/中转节点) │                │ (出口节点)    │                │(Website) │

└──────────┘                └─────────────┘                └─────────────┘                └──────────┘

```

  

-   **客户端**: 您的个人电脑或设备。

-   **服务器 A (入口节点)**: 直接与客户端连接的服务器，负责将流量转发给服务器 B。这台服务器的 IP 可以被 CDN 隐藏。

-   **服务器 B (出口节点)**: 最终访问互联网的服务器，其 IP 地址将是您对外显示的地址。

  

---

  

## 准备工作

  

1.  **两台境外服务器**:

    *   服务器 A (入口): 配置不要求太高，但网络质量要好。

    *   服务器 B (出口): 建议选择带宽较大���地理位置合适的服务器。

    *   确保两台服务器都开放了必要的 TCP 和 UDP 端口。

2.  **一个域名**: 用于证书申请和域前置。例如 `yourdomain.com`。

3.  **CDN 服务**: 推荐使用 Cloudflare，免费套餐即可满足需求。

4.  **基础 Linux 操作知识**: 本教程以 Debian/Ubuntu 系统为例。

  

---

  

## 步骤一：搭建基础双跳板代理

  

在这一步，我们先实现最核心的代理链功能：`客户端 -> A -> B -> 互联网`。

  

### 1. 配置服务器 B (出口节点)

  

服务器 B 是流量的最终出口，它需要监听来自服务器 A 的连接，并将流量解密后发往互联网。

  

**a. 下载 Hysteria 2**

  

登录服务器 B，执行以下命令下载并解压最新版的 Hysteria 2。

  

```bash

# 创建安装目录

mkdir -p /etc/hysteria

cd /etc/hysteria

  

# 从 GitHub 下载最新版本 (请到 https://github.com/apernet/hysteria/releases 自行替换为最新版链接)

wget https://github.com/apernet/hysteria/releases/download/v2.3.0/hysteria-linux-amd64 -O hysteria

chmod +x hysteria

```

  

**b. 创建配置文件**

  

创建一个配置文件 `config.yaml`。

  

```bash

nano /etc/hysteria/config.yaml

```

  

写入以下内容：

  

```yaml

# 服务器 B (出口节点) 配置文件: /etc/hysteria/config.yaml

  

# 监听来自服务器 A 的端口

listen: :443 # 您可以自定义端口，例如 :30000

  

# TLS 证书配置 (自动申请)

tls:

  acme:

    domains:

      - b.yourdomain.com # 解析到服务器 B IP 的域名

    email: your-email@gmail.com

  

# 认证配置 (设置一个强密码)

auth:

  type: password

  password: "YOUR_STRONG_PASSWORD_HERE" # 用于 A -> B 的连接认证

```

  

**说明**:

*   `listen`: Hysteria 监听的地址和端口。建议使用 443 端口，伪装成 HTTPS 流量。

*   `tls.acme`: Hysteria 会使用 ACME 协议自动从 Let's Encrypt 申请并续签 TLS 证书。请确保 `b.yourdomain.com` 的 A 记录已正确指向服务器 B 的 IP 地址。

*   `auth.password`: 用于服务器 A 连接服务器 B 时的认证密码。

  

**c. 启动服务并设置防火墙**

  

```bash

# 开放防火墙端口 (以 ufw 为例)

ufw allow 443/tcp

ufw allow 443/udp

ufw reload

  

# 启动 Hysteria 服务

./hysteria server -c ./config.yaml

```

  

为了让服��在后台持续运行，建议使用 `systemd`。

  

```bash

# 创建 systemd 服务文件

nano /etc/systemd/system/hysteria.service

```

  

写入以下内容：

  

```ini

[Unit]

Description=Hysteria 2 Service (Server B)

After=network.target

  

[Service]

Type=simple

ExecStart=/etc/hysteria/hysteria server -c /etc/hysteria/config.yaml

WorkingDirectory=/etc/hysteria

Restart=on-failure

RestartSec=5s

  

[Install]

WantedBy=multi-user.target

```

  

然后启动并设置开机自启：

  

```bash

systemctl daemon-reload

systemctl enable hysteria

systemctl start hysteria

systemctl status hysteria # 检查服务状态

```

  

### 2. 配置服务器 A (入口/中转节点)

  

服务器 A 接收来自客户端的连接，然后将流量转发给服务器 B。

  

**a. 下载 Hysteria 2**

  

在服务器 A 上重复下载步骤。

  

```bash

mkdir -p /etc/hysteria

cd /etc/hysteria

wget https://github.com/apernet/hysteria/releases/download/v2.3.0/hysteria-linux-amd64 -O hysteria

chmod +x hysteria

```

  

**b. 创建配置文件**

  

创建 `config.yaml` 文件。

  

```bash

nano /etc/hysteria/config.yaml

```

  

写入以下内容：

  

```yaml

# 服务器 A (入口节点) 配置文件: /etc/hysteria/config.yaml

  

# 监听来自客户端的端口

listen: :443

  

# TLS 证书配置 (自动申请)

tls:

  acme:

    domains:

      - a.yourdomain.com # 解析到服务器 A IP 的域名

    email: your-email@gmail.com

  

# 认证配置 (客户端连接到 A)

auth:

  type: password

  password: "ANOTHER_STRONG_PASSWORD" # 用于 Client -> A 的连接认证

  

# 核心：出站配置，将所有流量转发到服务器 B

outbounds:

  - type: remote

    server: b.yourdomain.com:443 # 服务器 B 的地址和端口

    auth: "YOUR_STRONG_PASSWORD_HERE" # A -> B 的连接认证密码

    # TLS 配置，用于验证服务器 B 的证书

    tls:

      sni: b.yourdomain.com # 必须与服务器 B 的证书域名一致

```

  

**说明**:

*   `listen`, `tls`, `auth`: 与服务器 B 类似，但用于客户端与服务器 A 之间的连接。确保 `a.yourdomain.com` 指向服务器 A 的 IP。

*   `outbounds`: 这是实现“跳板”的关键。它告诉服务器 A 将收到的所有流量都通过 `remote` 方式转发到 `b.yourdomain.com:443`。

*   `outbounds.auth`: 这里填写的是服务器 B ���置文件中设置的密码。

*   `outbounds.tls.sni`: Server Name Indication，用于验证服务器 B 的证书，必须填写服务器 B 证书对应的域名。

  

**c. 启动服务**

  

同样，为服务器 A 设置 `systemd` 服务并启动。步骤与服务器 B 完全相同，只需确保 `ExecStart` 和 `WorkingDirectory` 路径正确。

  

### 3. 配置客户端

  

现在，配置您的本地客户端连接到服务器 A。

  

**a. 下载 Hysteria 2**

  

在您的电脑上下载对应操作系统的 Hysteria 2 程序。

  

**b. 创建客户端配置文件 `config.yaml`**

  

```yaml

# 客户端配置文件: config.yaml

  

# 连接到服务器 A

server: a.yourdomain.com:443

auth: "ANOTHER_STRONG_PASSWORD" # Client -> A 的连接认证密码

  

# TLS 配置，用于验证服务器 A 的证书

tls:

  sni: a.yourdomain.com

  

# 本地 SOCKS5/HTTP 代理监听端口

listen: :1080

```

  

**c. 启动客户端**

  

在配置文件所在目录打开终端，运行：

  

```bash

./hysteria client -c ./config.yaml

```

  

启动成功后，Hysteria 会在本地 `1080` 端口开启一个 SOCKS5 和 HTTP 代理。将您的系统或浏览器代理设置为 `socks5://127.0.0.1:1080` 或 `http://127.0.0.1:1080`。

  

此时，访问 [ipinfo.io](https://ipinfo.io) 等网站，您应该能看到服务器 B 的 IP 地址。基础双跳板代理搭建完成！

  

---

  

## 步骤二：增强匿名性与安全性

  

为了防止流量被识别和干扰，我们为两段连接都加入流量混淆。

  

### 1. 更新服务器 B 配置

  

在服务器 B 的 `config.yaml` 中添加 `obfs` (混淆) 配置。

  

```yaml

# 服务器 B (出口节点) 配置文件

  

listen: :443

tls:

  # ... (不变)

auth:

  # ... (不变)

  

# 新增：流量混淆配置

obfs:

  type: "salamander" # 推荐的混淆类型

  password: "OBFS_PASSWORD_AB" # A -> B 连接的混淆密码

```

  

修改后，重启服务器 B 的 Hysteria 服务：`systemctl restart hysteria`。

  

### 2. 更新服务器 A 配置

  

服务器 A 需要在**出站 (outbounds)** 和 **入站 (inbounds)** 两处都进行修改。

  

```yaml

# 服务器 A (入口节点) 配置文件

  

listen: :443

tls:

  # ... (不变)

auth:

  # ... (不变)

  

# 新增：针对客户端连接的混淆

obfs:

  type: "salamander"

  password: "OBFS_PASSWORD_CLIENT_A" # Client -> A 连接的混淆密码

  

outbounds:

  - type: remote

    server: b.yourdomain.com:443

    auth: "YOUR_STRONG_PASSWORD_HERE"

    tls:

      sni: b.yourdomain.com

    # 新增：针对 A -> B 连接的混淆

    obfs:

      type: "salamander"

      password: "OBFS_PASSWORD_AB" # 必须与服务器 B 设置的混淆密码一致

```

  

重启服务器 A 的 Hysteria 服务：`systemctl restart hysteria`。

  

### 3. 更新客户端配置

  

在客户端 `config.yaml` 中添加 `obfs` 配置。

  

```yaml

# 客户端配置文件

  

server: a.yourdomain.com:443

auth: "ANOTHER_STRONG_PASSWORD"

tls:

  sni: a.yourdomain.com

listen: :1080

  

# 新增：流量混淆配置

obfs:

  type: "salamander"

  password: "OBFS_PASSWORD_CLIENT_A" # 必须与服务器 A 设置的混淆密码一致

```

  

重新启动客户端。现在，`客户端 -> A` 和 `A -> B` 两段链路都启用了流量混淆。

  

---

  

## 步骤三：实现域前置 (Domain Fronting)

  

域前置通过 CDN 隐藏服务器 A 的真实 IP，所有流量看起来都像是访问一个正常的网站（CDN 节点），从而有效抵抗基于 IP ��封锁。

  

### 1. 配置 CDN

  

以 Cloudflare 为例：

  

1.  登录 Cloudflare，选择您的域名 `yourdomain.com`。

2.  进入 "DNS" 记录页面。

3.  确保 `a.yourdomain.com` 的 A 记录存在，并且 "Proxy status"（代理状态）为 **Proxied** (橙色云朵)。这条记录指向服务器 A 的真实 IP。

4.  进入 "SSL/TLS" -> "Overview" 页面，将加密模式设置为 **Full (Strict)**。

5.  进入 "SSL/TLS" -> "Edge Certificates" 页面，开启 "Always Use HTTPS"。

  

### 2. 更新服务器 A 配置

  

由于 Cloudflare 会处理 TLS 握手，服务器 A 不再需要直接监听 443 端口或处理 TLS。我们将让它监听一个本地端口，并由反向代理（如 Nginx）将流量转发给它。

  

**a. 修改 Hysteria 配置**

  

修改服务器 A 的 `config.yaml`：

  

```yaml

# 服务器 A (入口节点) 配置文件 - 域前置版

  

# 监听一个本地非特权端口，不再需要 TLS

listen: 127.0.0.1:8080

  

# TLS 配置可以移除或注释掉

# tls:

#   ...

  

auth:

  # ... (不变)

obfs:

  # ... (不变)

outbounds:

  # ... (不变)

```

  

**b. 安装并配置 Nginx 作为反向代���**

  

```bash

# 安装 Nginx

apt update && apt install -y nginx

  

# 创建 Nginx 配置文件

nano /etc/nginx/sites-available/hysteria

```

  

写入以下内容：

  

```nginx

server {

    listen 443 ssl http2;

    listen [::]:443 ssl http2;

  

    server_name a.yourdomain.com; # 你的域名

  

    # 使用 Cloudflare 提供的证书，或者你自己的证书

    ssl_certificate /path/to/your/fullchain.pem; # 例如 Cloudflare Origin Certificate

    ssl_certificate_key /path/to/your/private.key;

  

    # 核心：将流量转发到 Hysteria 服务

    location / {

        proxy_pass http://127.0.0.1:8080; # 转发到 Hysteria 监听的本地地址

        proxy_http_version 1.1;

        proxy_set_header Host $host;

        proxy_set_header X-Real-IP $remote_addr;

        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_set_header Upgrade $http_upgrade;

        proxy_set_header Connection "upgrade";

    }

}

```

  

**说明**:

*   你需要为 Nginx 配置 SSL 证书。强烈建议使用 Cloudflare 的 **Origin Certificates**，这可以确保从 Cloudflare 到你服务器的流量也是加密的。在 Cloudflare "SSL/TLS" -> "Origin Server" 页面创建证书，然后将证书和私钥文件上传到服务器 A 的 `/path/to/your/` 目录下。

*   启用该配置：`ln -s /etc/nginx/sites-available/hysteria /etc/nginx/sites-enabled/`

*   测试并重启 Nginx：`nginx -t && systemctl restart nginx`

*   重启 Hysteria 服务：`systemctl restart hysteria`

  

### 3. 更新客户端配置

  

客户端现在需要通过 CDN 连接，同时使用 `sni` 字段来指定真实的服务器域名以通过 TLS 验证。

  

```yaml

# 客户端配置文件 - 域前置版

  

# server 地址现在是 CDN 域名

server: a.yourdomain.com:443

auth: "ANOTHER_STRONG_PASSWORD"

obfs:

  type: "salamander"

  password: "OBFS_PASSWORD_CLIENT_A"

  

# TLS 配置现在至关重要

tls:

  # SNI 必须是你的域名，用于通过 Cloudflare 的验证

  sni: a.yourdomain.com

  # 如果你没有使用受信任的 CA 证书，可能需要关闭证书验证（不推荐）

  # insecure: true

  

listen: :1080

```

  

重启客户端。现在，你的流量路径是 `客户端 -> Cloudflare CDN -> 服务器 A (Nginx) -> 服务器 A (Hysteria) -> 服务器 B -> 互联网`。你的服务器 A 的真实 IP 被完全隐藏。

  

---

  

## 步骤四：配置反向代理 (可选)

  

如果你希望服务器 A 的 443 端口同时能提供一个正常的网站服务，以达到更好的伪装效果，可以修改 Nginx 配置。

  

假设你的网站文件放在 `/var/www/html`。

  

修改 `/etc/nginx/sites-available/hysteria`：

  

```nginx

server {

    listen 443 ssl http2;

    listen [::]:443 ssl http2;

  

    server_name a.yourdomain.com;

  

    ssl_certificate /path/to/your/fullchain.pem;

    ssl_certificate_key /path/to/your/private.key;

  

    # 默认显示网站内容

    root /var/www/html;

    index index.html;

  

    # 将特定路径的流量转发给 Hysteria

    # 这个路径需要和客户端配置中的 `server` 字段匹配

    location /YourSecretPath {

        proxy_pass http://127.0.0.1:8080;

        proxy_http_version 1.1;

        # ... (其他 proxy_set_header 指令不变)

    }

}

```

  

然后修改客户端 `config.yaml` 的 `server` 字段：

  

```yaml

# 客户端配置文件 - 反向代理版

server: a.yourdomain.com:443/YourSecretPath

# ... 其他配置不变

```

  

这样，直���访问 `https://a.yourdomain.com` 会看到你的网站，而 Hysteria 客户端会通过 `https://a.yourdomain.com/YourSecretPath` 连接，实现了服务伪装。

  

---

  

## 总结与验证

  

至此，您已成功搭建了一个功能完备、高匿名的双跳板代理。

  

*   **验证 IP**: 开启代理后，访问 `ipinfo.io`，应显示服务器 B 的 IP。

*   **验证延迟**: 使用 `ping` 或其他工具测试延迟。由于经过了中转，延迟会高于直连。

*   **检查日志**: `journalctl -u hysteria -f` (在服务器上) 可以实时查看 Hysteria 的日志，用于排查连接问题。

  

请务必使用强密码，并妥善保管您的配置文件。