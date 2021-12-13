# gitlab

## gitlab安装

### linux命令行安装

#### 安装依赖项

```
sudo yum install -y curl policycoreutils-python openssh-server
sudo systemctl enable sshd
sudo systemctl start sshd
```

#### 防火墙设置

```
- 关闭firewall服务
  sudo systemctl stop firewalld
  sudo systemctl disable firewalld
- 安装iptables服务
- yum install -y iptables-services
  systemctl enable iptables
  systemctl start iptables
  sudo systemctl enable iptables
- 设置防火墙规则
  iptables -I INPUT -s 127.0.0.1 -d 127.0.0.1 -j ACCEPT
  iptables -I INPUT -s x.x.x.x -p tcp --dport 80 -j ACCEPT
  iptables -I INPUT -s x.x.x.x -p tcp --dport 80 -j ACCEPT
  iptables -A INPUT 3 -p tcp --dport 22 -j ACCEPT
  iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
  iptables -A INPUT -j REJECT
  service iptables save
```

#### 安装Postfix以发送通知电子邮件

```
- sudo yum install postfix
- sudo systemctl enable postfix
- sudo systemctl start postfix
```

#### 添加GitLab软件包存储库并安装软件包

访问gitlab官网，企业版地址：https://packages.gitlab.com/gitlab/gitlab-ee，选择自己需要的版本进入。复制右上角的

```
curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash执行
```

#### 安装GitLab包

在上述版本包页面，拷贝Install中的命令并执行：

```
sudo yum install gitlab-ee-12.10.14-ee.0.el7.x86_64
```

#### 安装后需要手动配置gitlab的访问地址

**修改配置文件**

vim /etc/gitlab/gitlab.rb

将external_url "http://gitlab.example.com" 修改为指定的地址

如 external_url "http://192.168.xx.xx"

**配置邮件服务，创建用户后邮件通知**

```
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.office365.com"
gitlab_rails['smtp_port'] = 587
gitlab_rails['smtp_user_name'] = "XXXX@quectel.com"
gitlab_rails['smtp_password'] = "**"
gitlab_rails['smtp_domain'] = "quectel.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = false
gitlab_rails['gitlab_email_from'] = "XXXX@quectel.com"
```

运行sudo gitlab-ctl reconfigure以使更改生效

#### 访问服务

服务地址为步骤5中external_url的值，第一次访问需要设置密码，该密码的用户是root（即管理员）

#### 服务启停

- 启动：sudo gitlab-ctl start
- 停止：sudo gitlab-ctl stop
- 重启：sudo gitlab-ctl restart

### docker安装

```
docker pull gitlab/gitlab-ce[:TAG]
docker run -d -p 443:443 -p 80:80 -p 222:22 --name gitlab --restart always -v /data/gitlab/config:/etc/gitlab -v /data/gitlab/logs:/var/log/gitlab -v /data/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce[:TAG]
```

## gitlab-runner安装

### linux命令行安装

```
# 清华源下载rpm包
https://mirrors.tuna.tsinghua.edu.cn/gitlab-runner/yum/el7/gitlab-runner-xx.xx.x86_64.rpm
# 安装
rpm -ivh gitlab-runner-xx.xx.x86_64.rpm
systemctl start gitlab-runner
systemctl enable gitlab-runner
```

### docker安装

```
# 拉取镜像
docker pull gitlab/gitlab-runner[:TAG]
# 运行镜像
docker run -itd --name gitlab-runner -v /data/gitlab-runner/config:/etc/gitlab-runner gitlab/gitlab-runner
```

## gitlab-runner注册

### gitlab-runner类型

- shared：运行整个平台项目的作业（gitlab）
- group：运行特定group下的所有项目的作业（group）
- specific：运行指定的项目作业（project）
- locked：无法运行项目作业
- paused：不会运行作业

### gitlab-runner注册

#### 交互式

运行gitlab-runner register，后续按要求填写

#### 非交互式

运行下列命令

```shell
gitlab-runner register \
	--non-interactive \
	--executor "shell" \
	--url "http://xx.xx.xx.xx:xx" \
	--registration-token "xxxxxxx" \
	--description "build node" \
	--tag-list "build,deploy" \
	--locked="false" \
	--access-level="not_protected"
```

