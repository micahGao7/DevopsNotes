# 1、Zabbix部署

## 1.1 apt安装zabbix

### 1.1.1 install zabbix repository

```shell
wget https://repo.zabbix.com/zabbix/5.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_5.0-1+bionic_all.deb
dpkg -i zabbix-release_5.0-1+bionic_all.deb
apt update
```

### 1.1.2 安装zabbix server，web前端，agent

```shell
apt install zabbix-server-mysql zabbix-frontend-php zabbix-nginx-conf zabbix-agent
```

### 1.1.3 创建初始数据库

#### 1.1.3.1 安装mariadb

```shell
# 安装mariadb
apt install mariadb-client-10.1 mariadb-server -y
# 配置安全选项
mysql_secure_installation
```

#### 1.1.3.2 初始化

```
mysql -uroot -p
password
mysql> create database zabbix character set utf8 collate utf8_bin;
mysql> create user zabbix@localhost identified by 'password';
mysql> grant all privileges on zabbix.* to zabbix@localhost;
mysql> quit;
```

#### 1.1.3.3 导入初始架构和数据

```shell
zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix
```

#### 1.1.3.4 为zabbix server配置数据库

```shell
# 编辑配置文件/etc/zabbix/zabbix_server.conf
DBPassword=zabbix
```

#### 1.1.3.5 为zabbix前端配置php

```shell
# 编辑配置文件/etc/zabbix/nginx.conf
listen 80;
server_name xxxxx;
# 编辑配置文件/etc/zabbix/php-fpm.conf
php_value[date.timezone] = Asia/Shanghai
```

#### 1.1.3.6 启动zabbix server和agent进程

```
# 启动Zabbix server和agent进程，并为它们设置开机自启
systemctl restart zabbix-server zabbix-agent nginx php7.2-fpm
systemctl enable zabbix-server zabbix-agent nginx php7.2-fpm
```

## 1.2 web界面中文菜单环境

### 1.2.1 中文选项无法选择

由于Ubuntu Server系统未安装中文语言环境，所以User settings --> language无法选择中文

### 1.2.2 Ubuntu系统安装中文语言环境

```shell
# 安装简体中文语言环境
apt-get install language-pack-zh*

# 增加中文语言环境变量
echo LANG="zh_CN.UTF-8" >> /etc/environment

# 重新设置本地配置
dpkg-reconfigure locales --> 选择zh_CN.UTF-8
```

### 1.2.2 重启nginx

## 1.3 监控项乱码问题

### 1.3.1 在Windows拷贝字体

在windows上 **控制面板-->字体-->楷体 常规**，然后将字体拷贝到出来

### 1.3.2 上传字体到zabbix web目录

将windows字体文件(simkai.ttf)上传至**/usr/share/zabbix/assets/fonts**目录，并修改文件权限：

chown zabbix. /usr/share/zabbix/assets/fonts/*

### 1.3.3 修改zabbix配置文件调用新字体

```shell
# 修改文件/usr/share/zabbixinclude/defines.inc.php
define('ZBX_GRAPH_FONT_NAME',           'simkai');
define('ZBX_FONT_NAME', 'simkai');
```



