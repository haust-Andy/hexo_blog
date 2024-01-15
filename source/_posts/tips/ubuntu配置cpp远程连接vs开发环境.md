---
title: ubuntu配置cpp远程连接vs开发环境
date: 2024-01-11 
tags: Ubuntu
categories: 环境配置
---
### 1.ubuntu 首次设置root密码
切换到root用户身份：sudo -i

系统会要求输入当前用户的密码，然后按Enter确认。

接下来，使用以下命令设置root用户的新密码：passwd root

系统将提示您输入新的root密码两次，并显示成功更改密码的消息。

现在，您已经完成了对root用户的首次设置。
### 2.ubuntu 配置ssh连接
安装net服务
```
apt install net-tools
sudo apt update
sudo apt install openssh-server
```
修改配置文件
```
apt install vim
sudo vim
命令：sudo vim /etc/ssh/sshd_config
找到 #PermitRootLogin prohibit-password
改为 PermitRootLogin yes
重启ssh服务
sudo systemctl restart ssh
```
### 3.虚拟机设置静态ip
nat模式 手动分配ip  前三个四字节与主机相同 后一个随便，与主机、网关不同即可。网关前三个四字节与主机相同后一个四字节为2。例
192.168.187.1   主机ip
192.168.187.2   虚拟机网关
192.168.187.3   虚拟机手动设置ip
### 4.ubuntu安装mysql并配置 
```
apt-get upgrade #更新
apt-get update  
#安装mysql
apt-get install mysql-server

```

#### 设置root密码
```
将root用户的密码认证方式从"mysql_native_password"更改为"caching_sha2_password"，则需要使用以下命令：
ALTER USER 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY '<新密码>';
flush privileges;
update user set host='%' where user='root';
flush privileges;

```
#### 配置可访问ip
```
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
在 bind-address=127.0.0.1 前加 # 注释
保存关闭文件
重启mysql，可能需要填入系统认证信息
service mysql restart
配置MySQL开机自启
sudo update-rc.d mysql defaults

安装mysqlclientc++(C++的一个链接器)
sudo apt-get install libmysqlcppconn-dev

```


参考
https://blog.csdn.net/weixin_43571107/article/details/129271752