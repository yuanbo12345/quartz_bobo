
> [!NOTE] 前提
> 1. 最好使用clash_linux配置好代理链接
> 2. clashtun on全局代理 更新，安装各种包
> 3. 全局代理下还可以更好的使用google翻译等其他LLM_API_KEY

# api_key
## Free LLm_api_key combination


# Pdf2zh install turiol
## Using docker install
![[image-233.png]]

## Using GUI install
![[image-234.png]]

### A little of Attention 
1. 全程使用clash_linux进行安装和包管理以及翻译过程
2. 找机会改成中文界面版本
3. 采用云服务器的话，注意安全组放行端口以及nat端口内外转换
安全组放行设置
```bash
ip 0.0.0.0/0 端口 7860 (如果有nat转换的话就要放行面向外网的端口)
上面教程中说的7860是内网端口，可以通过nat转换设置成一样的端口、
协议：tcp和udp协议
```

