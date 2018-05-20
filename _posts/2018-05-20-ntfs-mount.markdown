---
layout:     post
title:      "使用ntfs-3g 挂载NTFS分区"
subtitle:   "Linux下挂载NTFS分区"
date:       2018-05-20
author:     "kikyoar"
header-img: "img/post-bg-linux-version.jpg"
tags:
    - Linux
---

打开ntfs-3g的下载站点，将最新稳定版下载到CentOS，执行以下命令安装：  

**[ntfs-3g-1.2918.tgz](http://down1.chinaunix.net/distfiles/ntfs-3g-1.2918.tgz)**   

> 1、编译安装ntfs-3g:  

	./configure 
	make 
	make install  
	
> 2、fdisk -l查看磁盘设备点：  
> 3、挂载NTFS分区： 

	mount -t ntfs-3g /dev/sda6 /mnt/ntfs -o force    

```  

如果需要开机自启动挂载，可以在/etc/fstab文件结尾添加需要挂载的NTFS盘
添加命令如下：
/dev/sda1 /mnt/C ntfs-3g defaults 0 0
如果有多个盘挂载，就多加几行喽～
本人的如下。
LABEL=/                 /                       ext3    defaults        1 1
tmpfs                   /dev/shm                tmpfs   defaults        0 0
devpts                  /dev/pts                devpts gid=5,mode=620 0 0
sysfs                   /sys                    sysfs   defaults        0 0
proc                    /proc                   proc    defaults        0 0
/dev/sda8               swap                    swap    defaults        0 0
/dev/sda1               /mnt/C                  ntfs-3g defaults        0 0
/dev/sda5               /mnt/D                  ntfs-3g defaults        0 0
/dev/sda6               /mnt/E                  ntfs-3g defaults        0 0
/dev/sda7               /mnt/F                  ntfs-3g defaults        0 0

```
