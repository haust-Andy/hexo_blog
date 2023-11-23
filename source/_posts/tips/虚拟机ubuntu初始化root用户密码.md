---
title: ubuntu一些配置信息
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