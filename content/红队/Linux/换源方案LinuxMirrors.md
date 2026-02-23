# 1. 先把脚本下载到本地
curl -fsSL https://raw.githubusercontent.com/SuperManito/LinuxMirrors/main/ChangeMirrors.sh -o /tmp/ChangeMirrors.sh

# 2. 再运行（可不加 sudo，脚本内部会自己提权）
bash /tmp/ChangeMirrors.sh --edu

接下来就会自动进入换源设置，按下Enter选择即可

