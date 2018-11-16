---
layout:     post
title:      "Linux查看服务器配置信息"
subtitle:   "Linux下物理服务器查询命令解析"
date:       2018-11-16
author:     "kikyoar"
header-img: "img/post-bg-linux-version.jpg"
tags:
    - Linux
--- 

## CPU  

**总核数 = 物理CPU个数 X 每颗物理CPU的核数**     
**总逻辑CPU数 = 物理CPU个数 X 每颗物理CPU的核数 X 超线程数**  

**查看物理CPU个数**  
`cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l`

**查看每个物理CPU中core的个数(即核数)**  
`cat /proc/cpuinfo| grep "cpu cores"| uniq`

**查看逻辑CPU的个数**
`cat /proc/cpuinfo| grep "processor"| wc -l`

**查看CPU信息（型号）**  
`cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c`  


## 内存

**查看内存槽及内存条**  

`sudo dmidecode -t memory`  

**查看内存的插槽数,已经使用多少插槽.每条内存多大**

`sudo dmidecode -t memory | grep Size`

**查看服务器型号、序列号**

`sudo dmidecode | grep "System Information" -A9 | egrep "Manufacturer|Product|Serial"`  

## 硬盘  

查看硬盘大小  

`fdisk -l | grep Disk`

查看硬盘和分区分布  

`lsblk`

## 查看服务器型号  

`grep 'DMI' /var/log/dmesg`  

## 查看主板型号
`dmidecode |grep -A16 "System Information$"`  


## 查看物理网卡
`lspci | grep Ethernet` 

