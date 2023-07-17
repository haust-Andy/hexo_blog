---
title: 实现vmware虚拟机和主机之间的文件传输
tags: vmware
categories: 环境配置
---

# 1.设置共享文件夹
启动虚拟机→点击虚拟机→点击设置（可以直接快捷键：Ctrl+D）
  在终端命令行输入如下命令：vmware-hgfsclient
  会显示共享文件夹
  输入命令：cd /mnt/hgfs  可进入目录查看到虚拟机中共享文件夹所在的位置。
# 2.WinSCP

https://blog.csdn.net/weixin_44380309/article/details/125876155