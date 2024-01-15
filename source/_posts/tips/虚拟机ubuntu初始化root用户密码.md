---
title: ubuntu的一些配置命令
date: 2023-06-26 
tags: ubuntu
categories: 环境配置
---

# 虚拟机ubuntu初始化root用户密码
第一步：Ctrl+Alt+T打开终端 <br>
第二步：输入命令 sudo passwd <br>
root <br>
输入当前用户密码 <br>
输入所想要设置的root密码 <br>
再次输入root密码 <br>
第三步：输入命令su <br>
输入root密码即切换至root用户模式下 <br>
# ssh 配置
`sudo apt install ssh`          
`sudo systemctl status ssh`

`sudo apt install g++ gdb make cmake`
# Ubuntu如何使用root用户远程登陆
这是Ubuntu系统处于安全性的考虑；

我们不能使用root用户远程连接Ubuntu系统的原因是因为，系统内的/etc/ssh/sshd_config配置文件内PermitRootLogin这一行为no决定了我们无法使用root用户远程登陆，所以我们只需要更改这个配置文件的内容

输入cat /etc/ssh/ssh_config命令我们可以查看这个配置文件

可以看到PermitRootLogin yes  （这里我已经从no改成yes了）

三、具体操作

1、首先我们先启动VMware里面的Ubuntu系统

使用Xshell远程连接使用普通用户登陆

2、我们先为root用户设置一个登陆密码

输入sudo passwd root

提示我们输入当前用户的密码；

之后我们给root用户设置密码；

出现passwd: password updated successfully说明成功给root用户设置了密码

3、修改配置文件

就像我们之前提到的一样，无法远程登陆是因为配置文件的设定，所以我们应该修改配置文件实现root用户的远程登陆；

输入vim /etc/ssh/sshd_config使用vim文本编辑器修改sshd_config这个配置文件

敲键盘i键进入插入模式，利用上下方向键找到PermitRootLogin的位置修改no为yes

敲键盘上的esc键退出修改，英文输入法下输入冒号键输入wq退出

4、重新启动ssh服务

上面我们已经完成了修改，此时输入命令 service ssh restart重启ssh服务即可

使用 su - root命令切换到root用户登陆

输入密码即可成功登陆

可以看到我们当前使用root用户成功登陆
————————————————
版权声明：本文为CSDN博主「蜜糖伴午茶」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/m0_63276793/article/details/134038194
