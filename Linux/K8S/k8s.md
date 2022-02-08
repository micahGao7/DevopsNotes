# 高可用集群搭建

## 命令方式

### 辅助环境

#### **docker安装**

```shell
编辑软件安装脚本docker_install.sh
#!/bin/bash
# 安装Docker环境
# 安装依赖环境
apt-get update
apt-get install ca-certificates curl gnupg lsb-release -y

#配置软件源
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
apt-get update

#安装软件
apt-get install docker-ce docker-ce-cli containerd.io -y

#加速器配置（k8s node节点中无需此配置）
echo '{"registry-mirrors": ["https://5sdlkupt.mirror.aliyuncs.com"],"insecure-registries": ["10.0.0.19:80"]}' > /etc/docker/daemon.json
#重启服务
systemctl restart docker
systemctl enable docker
```

**定制docker的启动方式**

```shell
# vim /etc/docker/daemon.json
{
	"exec-opts": ["native.cgroupdriver=systemd"]
}
# 重启docker
systemctl daemon-reload
systemctl restart docker
```

**harbor部署**

```shell
安装docker-compose
apt-get install docker-compose -y

下载harbor-offline-installer压缩包
wget https://github.com/goharbor/harbor/releases/download/v2.3.2/harbor-offline-installer-v2.x.x.tgz

解压
tar -zxvf harbor-offline-installer-v2.x.x.tgz -C /usr/local

加载镜像
docker load -i /usr/local/harbbor.v2.x.x.tar.gz

备份配置
cp harbor.yml.tmpl harbor.yml

修改配置
	hostname: 10.0.0.9
	# 注释HTTPS相关部分
	harbor_admin_password: Harbor12345
	data_volume: /data/harbor

配置harbor
./prepare

启动harbor
./install.sh

编写服务启动文件
# vim /lib/systemd/system/harbor.service
[Unit]
Description=Harbor
After=docker.service systemd-networkd.service systemd-resolved.service
Requires=docker.service
Documentation=http://github.com/vmware/harbor

[Service]
Type=simple
Restart=on-failure
RestartSec=5
#需要注意harbor的安装位置
ExecStart=/usr/bin/docker-compose -f /usr/local/harbor/docker-compose.yml up
ExecStop=/usr/bin/docker-compose -f /usr/local/harbor/docker-compose.yml down

[Install]
WantedBy=multi-user.target

加载服务配置文件
systemctl daemon-reload
systemctl enable harbor
```

#### **swap禁用、网络转发**

```shell
# 脚本方式
#!/bin/bash
# 禁用swap
swapoff -a
sed -i 's/.*swap.*/#&/' /etc/fstab
cat >> /etc/sysctl.d/k8s.conf << EOF
vm.swappiness=0
EOF
sysctl -p /etc/sysctl.d/k8s.conf

# 打开网络转发
cat >> /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
modprobe br_netfilter
modprobe overlay
sysctl -p /etc/sysctl.d/k8s.conf
```

#### **主机名解析配置**

```
# 编辑/etc/hosts 文件
10.0.0.12 master1.micah.com master1
10.0.0.13 master2.micah.com master2
10.0.0.14 master3.micah.com master3
10.0.0.15 node1.micah.com node1
10.0.0.16 node2.micah.com node2
10.0.0.17 ha1.micah.com ha1
10.0.0.18 ha2.micah.com ha2
10.0.0.19 register.micah.com register
```

#### 高可用软件安装

安装软件

```
apt install keeplived haproxy -y
```

keepalived配置

```shell
# 准备配置文件
cp /usr/share/doc/keepalived/samples/keepalived.conf.vrrp /etc/keepalived/keepalived.conf

# 编辑配置文件
# master配置文件
vim /etc/keepalived/keepalived.conf
global_defs {
   router_id ha-1
}

vrrp_script chk_haproxy {
    script "/etc/keepalived/check_haproxy.sh" # haproxy状态监控脚本
    interval 2
    weiht 2
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100
    authentication {
        auth_type PASS
        auth_pass 1111
    }

    track_script {
        chk_haproxy
    }

    virtual_ipaddress {
        10.0.0.200 dev eth0 label eth0:1
    }
}

# backup配置文件
global_defs {
   router_id ha-2
}

vrrp_script chk_haproxy {
    script "/etc/keepalived/check_haproxy.sh"  # haproxy状态监控脚本
    interval 2
    weiht 2
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 90
    authentication {
        auth_type PASS
        auth_pass 1111
    }

    track_script {
        chk_haproxy
    }

    virtual_ipaddress {
        10.0.0.200 dev eth0 label eth0:1
    }
}
```

haproxy配置

```shell
# 修改配置文件
vim /etc/haproxy/haproxy.cfg
...
listen status
    bind 10.0.0.200:9999
    mode http
    log global
    stats enable
    stats uri /haproxy-stats
    stats auth haadmin:123456

listen k8s-api-6443
	bind 10.0.0.200:6443
    mode tcp
    server master1  10.0.0.12:6443 check inter 3s fall 3 rise 5
    server master2  10.0.0.13:6443 check inter 3s fall 3 rise 5
    server master3  10.0.0.14:6443 check inter 3s fall 3 rise 5
    
# 编写haproxy状态监控脚本
vim /etc/keepalived/check_haproxy.sh
proxy_status=$(ps -C haproxy --no-header | wc -l)
if [ ${haproxy_status} -eq 0 ];then
    systemctl start haproxy
    sleep 3
    if [ $(ps -C haproxy --no-header | wc -l) -eq 0 ];then
    	killall keepalived
    fi
fi
```

#### **kubeadm配置**

kubeadm软件源的配置

```
apt-get update
apt-get install apt-transport-https
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add -
cat >> /etc/apt/sources.list.d/kubernetes.list <<EOF
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF
apt-get update

# 查看支持的软件版本
apt-cache madison kubeadm
```

#### 命令补全

将相关环境配置放到当前用户的环境文件中

```
echo "source <(kubectl completion bash)" >> ~/.bashrc
```

### 集群安装

#### master部署

安装软件

```
# 安装kubelet、kubeadm、kubectl
apt-get install kubelet=1.22.1-00 kubeadm=1.22.1-00 kubectl=1.22.1-00 -y

注意：
	kubeadm 将node节点加载到master集群中的命令
	kubelet 用于采集当前节点信息，推送给master的服务
	kubectl 用于管理k8s集群的管理命令，在node节点上不用，一般不安装
```

**master1节点集群初始化**

```
kubeadm init --kubernetes-version=1.22.1 \
--apiserver-advertise-address=10.0.0.12 \
--control-plane-endpoint=10.0.0.200 \
--apiserver-bind-port=6443 \
--image-repository 10.0.0.19:80/google_containers \
--service-cidr=10.96.0.0/12 \
--pod-network-cidr=10.244.0.0/16 \
--ignore-preflight-errors=Swap
```

**赋予权限**

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

**网络容器**

````
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl apply -f kube-flannel.yml
````

#### 添加master节点

**软件安装**

```
# 安装kubelet、kubeadm、kubectl
apt-get install kubelet=1.22.1-00 kubeadm=1.22.1-00 kubectl=1.22.1-00 -y
```

**节点增加**

````
# 在集群初始化节点上生成通信认证的key
kubeadm init phase upload-certs --upload-certs

# 将新的主节点添加到master集群中
kubeadm join 10.0.0.200:6443 --token h0jk4r.ncjymstn1qfi9isy --discovery-token-ca-cert-hash sha256:9f1d372e68aad7a3599e43a9221232fe380ebe0e0532afe8640f4382f29353bf --control-plane --certificate-key 937657b40b35b273b75bee9ea00a4168d99f3f3dd3325461984a7c75e0d3282d
````

**异常还原**

```
# 如果节点添加失败，则执行下面两条命令可以让环境还原
kubeadm reset
rm -rf /etc/kubernetes
```

#### 添加node节点

安装软件

```
# 安装kubelet、kubeadm
apt-get install kubelet=1.22.1-00 kubeadm=1.22.1-00 -y
```

添加节点

```
# node docker无需配置镜像仓库地址，默认从master同步组件镜像
kubeadm join 10.0.0.200:6443 --token h0jk4r.ncjymstn1qfi9isy --discovery-token-ca-cert-hash sha256:9f1d372e68aad7a3599e43a9221232fe380ebe0e0532afe8640f4382f29353bf
```

## 配置文件方式

