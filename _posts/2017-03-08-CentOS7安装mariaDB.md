---
title: CentOS7安装mariaDB
categories:
 - DevOps
tags: DevOps
---

centos7默认安装的是mariadb，所以安装命令与之前安装mysql略有不同。
```
yum install -y mariadb-server mariadb #安装服务端和客户端
systemctl start mariadb.service #启动服务 /usr/lib/systemd/system/mariadb.service
systemctl enable mariadb #设置开机启动
```
设置用户密码
```
set password for 'root'@'localhost' =password('root'); #设置密码
```
远程连接设置
```
grant all privileges on *.* to root@'%'identified by 'root';
```
myriadb的配置文件在/etc/my.cnf，可以设置编码，打开日志等。
