# winsows安装教程
1. 首先就是要通过powershell以命令行形式进行安装，以管理员方式运行Powershell
2. 从nodejs官网下载安装nodejs，一定要添加到环境变量，安装完成后重启电脑，重启终端
3. 可以创建一个文件夹，在文件夹中安装
```bash
mkdir -f Geminicli
cd Geminicli
```
4. 接着验证nodejs环境是否以安装成功
```bash
node -v
npm -v
npx -v
```
5. 如果这三个命令都回显，显示各自的版本号，好说基本环境安装成功，下面安装geminicli工具
```bash
npx https://github.com/google-gemini/gemini-cli // 通过npx包管理器安装
// 方法2
npm install -g @google/gemini-cli
gemini // 也是唤醒终端的命令
```
6.下载完毕后会自动打开gemini界面，上下键选择主题和Api_key登录
```bash
export GEMINI_API_KEY="YOUR_API_KEY" // linux命令

$env:GEMINI_API_KEY="YOUR_API_KEY" // windows命令
```
7. 设置代理，windows直接打开clash或者随意代理软件，规则模式即可
```bash
// 设置本地监听代理通信端口即可，示例为clash
$env:HTTP_PROXY="http://127.0.0.1:7890"
$env:HTTPS_PROXY="http://127.0.0.11:7890"
```
8. Linux设置代理可以采用clash_linux项目即可，只需输入订阅
9. 注意关于clash的设置![[image-225.png]]
