# 搭建MediaWiki在OpenCloudOS系统上

## 安装操作系统

* 请参考[OpenCloudOS安装](https://github.com/nmknm14/OpenCloudOS_install_steps)进行系统安装。

* 安装完成后请使用终端工具（如：Xshell、PuTTY等）连接服务器，进行后续操作。
* 查看系统版本：
  ```bash
  cat /etc/os-release
  ``` 
* 安装完成后更新系统：

  ```bash
  sudo dnf check-update && sudo dnf upgrade -y && sudo dnf clean all
  ```

## 关闭SELinux
```bash
进入目录: /etc/selinux/config
将 SELINUX=enforcing  更改为 SELINUX=disabled，然后保存文件。
根据您的操作系统和分配，您可能需要重启机器(reboot)，使变更生效。
在命令行中输入 sestatus，查看状态。
```
## 固定IP地址
* 查看网卡信息：
```bash
nmcli device status
```
* 设置静态IP地址：
```bash
nmcli con mod "有线连接 1" ipv4.addresses 192.168.0.104/24
nmcli con mod "有线连接 1" ipv4.gateway 192.168.0.1
nmcli con mod "有线连接 1" ipv4.dns "192.168.0.1"
nmcli con mod "有线连接 1" ipv4.method manual
```
* 重启网络服务：
```bash 
nmcli con down "有线连接 1"
nmcli con up "有线连接 1"
``` 
* 查看IP地址：
```bash 
hostname -I
```
## 放行80和443端口或者放行http和https服务

```bash 
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```
* 查看端口是否放行
```bash
sudo firewall-cmd --list-all
```
## 安装unzip软件
```bash
sudo dnf install unzip
```

## 安装nginx

```bash
sudo dnf install nginx
```
* 创建网站文件夹
```bash
sudo mkdir -p /var/www/wiki
```
* 下载并解压MediaWiki到网站文件夹
```bash
sudo unzip mediawiki-1.39.4.zip -d /var/www/wiki
``` 
* 将文件移动到合适位置
```bash
sudo mv /var/www/wiki/mediawiki-1.39.4/* /var/www/wiki/
``` 
* 修改nginx配置文件
```bash
sudo vim /etc/nginx/nginx.conf
```
* 一个参考
```
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log notice;
pid /run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    keepalive_timeout   65;

    include /etc/nginx/conf.d/*.conf;

    server {
        listen 80;
        server_name _;
        root /var/www/wiki;
        index index.php index.html;

        location / {
            try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
            fastcgi_pass unix:/run/php-fpm/www.sock;
            fastcgi_index index.php;
            include /etc/nginx/fastcgi.conf;
        }
    }
}

```
* 给予nginx访问网站文件夹权限
```bash
sudo chown -R nginx:nginx /var/www/wiki
``` 
* 启动nginx服务
```bash
sudo systemctl start nginx
sudo systemctl enable nginx
``` 
## 安装php

```bash
sudo dnf install php
```
### 查看php已经有的拓展

```bash
php -m
```
*[MediaWiki需要的PHP扩展](https://www.mediawiki.org/wiki/PHP_configuration/zh)*
### 安装缺少的php扩展

```bash
sudo dnf install php-mysqli php-apcu php-gd php-intl
```
### 开启php-fpm服务

```bash 
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
``` 
### 修改php-fpm配置文件

```bash
sudo vim /etc/php-fpm.d/www.conf
``` 
* 将user和group的值改为nginx
```bash
user = nginx
group = nginx
```
* 重启php-fpm服务
```bash
sudo systemctl restart php-fpm
``` 
*给予php-fpm访问网站文件夹权限
```bash
sudo chown -R nginx:nginx /var/www/wiki
``` 
## session权限设置

```bash
sudo chown -R nginx:nginx /var/lib/php/session
```
## 安装MariaDB
```bash
sudo dnf install mariadb-server
```
*[OpencloudOS官方手册安装MariaDB](https://docs.opencloudos.org/OCS/Typical_Application_Deployment/MariaDB_guide/)*

### 允许远程登陆
```bash
sudo mysql
```
```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY '19991207';
FLUSH PRIVILEGES;
EXIT;
```
