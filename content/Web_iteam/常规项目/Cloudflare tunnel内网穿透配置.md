# 安装教程
## 域名激活
1. 首先就是购买域名（可以国内阿里云购买，买最便宜的即可）

> [!NOTE] 注意
> 1. 购买域名之后一定要先新建个人模版，审核
> 2. 然后模板审核之后在申请域名实名认证审核

2. 然后登录自己的cloudflare网站，填入域名激活![[image-244.png]]
3. 根据官网显示会给你两个dns的解析地址，你需要打开你购买域名的控制台然后更改成cloud给的域名解析

> [!NOTE] 域名激活注意
> 1. 更改完dns解析之后，要等待大约30分钟，让解析上传到全球的根域名服务器，比较慢
> 2. 然后刷先cloud官网会提示，域名激活已成功就可以使用了

## 隧道配置
1. 接着就是配置好tunnel隧道
```bash
cloudflared tunnel create mytunnel // 这个tunnel就是你的隧道名称可以改
```
2. 创建配置文件
```bash
vi ~/.cloudflared/config.yml
```
config,yml
```bash
tunnel: mytunnel
credentials-file: /home/yuanbo/.cloudflared/mytunnel.json

ingress:
  - hostname: myhome.ounixiong.xyz
    service: http://localhost:8080
  - service: http_status:404

```


> [!NOTE] 配置文件命令解释
> 1. `hostname service`这两个变量是添加你自己的服务子域名以及子域名对应的内网服务的地址
> 2. 所以如果内网接入多个服务可以建立多个子域名，添加多个上来的两个变量
> 3. `service: http_status:404`这一行一直保留就可以了
> 4. tunnel：后面填写的一定要是你隧道的具体ID
> 5. mytunnel.json这个也是填你隧道具体ID的json名字

隧道查看命令
```bash
ls ~/.cloudflared

cloudflared tunnel list
```
![[image-245.png]]

## 子域名绑定到cloudflare上
```bash
cloudflared tunnel route dns mytunnel myhome.ounixiong.xyz
```
1. 首先mytunnel是你上面创建的隧道名字
2. myhome.ounixiong.xyz是你的服务内网子域名，上面配置文件写的
看隧道名字和ID的方法命令相同上面已给出

## 运行
```bash
cloudflared tunnel run mytunnel
```

1. 这个mytunnel是你的隧道名字或者ID都可以

## 速度慢提升方法
```yml
tunnel: YOUR_TUNNEL_ID
credentials-file: /root/.cloudflared/YOUR_TUNNEL.json

loglevel: info
protocol: h2mux
metrics: 127.0.0.1:4444
connections: 8  # 建议增加连接数提升吞吐

ingress:
  - hostname: your.example.com
    service: http://localhost:3001
  - service: http_status:404

```

## Hommar面板上设置内网多服务
1. 给每一个服务添加一个子域名之后
2. <mark style="background: #FF5582A6;">面板上新建应用，应用的URL链接就是你添加的子域名</mark>




