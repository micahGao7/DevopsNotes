

```
编辑软件安装脚本docker_insta11.sh
#!/bin/bash
# 安装Docker环境
# 安装依赖环境
apt-get update
apt-get install apt-transport-https ca-certificates curl gnupg 1sb-release -y

#配置软件源
curl -fsSL https://mirrors.tuna.tsinghua.edu.cn/docker-ce/1inux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
"deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu \
$(1sb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list >/dev/nu11
apt-get update

#安装软件
apt-get -y insta1l docker-ce docker-ce-cli containerd.io

#加速器配置
echo '{"registry-mirrors": ["https://5sdlkupt.mirror.aliyuncs.com"]，"insecure-registries": ["10.0.0.19:80"]}' > /etc/docker /daemon.json
#重启服务
systemct1 restart docker
systemctl enable docker
```

