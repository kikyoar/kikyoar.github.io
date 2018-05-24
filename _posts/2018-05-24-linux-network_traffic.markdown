---
layout:     post
title:      "详解Linux查看实时网卡流量的几种方式"
subtitle:   "实时网卡流量查询"
date:       2018-05-24
author:     "kikyoar"
header-img: "img/post-bg-linux-version.jpg"
tags:
    - Linux
---   

**1. sar -n DEV 1 2**   

sar命令包含在sysstat工具包中，提供系统的众多统计数据。其在不同的系统上命令有些差异，某些系统提供的sar支持基于网络接口的数据统计，也可以查看设备上每秒收发包的个数和流量  

	sar –n DEV 1 2  
命令后面1 2 意思是：每一秒钟取1次值，取2次  

DEV显示网络接口信息  

**另外，-n参数很有用，他有6个不同的开关：DEV、EDEV、NFS、NFSD、SOCK、ALL ，其代表的含义如下：**    

- DEV显示网络接口信息  
- EDEV显示关于网络错误的统计数据  
- NFS统计活动的NFS客户端的信息  
- NFSD统计NFS服务器的信息  
- SOCK显示套接字信息  
- ALL显示所有5个开关  

```
[sre@CDVM-213017031 ~]$ sar -n DEV 1 2
Linux 2.6.32-431.el6.x86_64 (CDVM-213017031)  05/04/2017  _x86_64_ (4 CPU)
	
08:05:30 PM  IFACE rxpck/s txpck/s rxkB/s txkB/s rxcmp/s txcmp/s rxmcst/s
08:05:31 PM  lo  0.00  0.00  0.00  0.00  0.00  0.00  0.00
08:05:31 PM  eth0 1788.00 1923.00 930.47 335.60  0.00  0.00  0.00
	
08:05:31 PM  IFACE rxpck/s txpck/s rxkB/s txkB/s rxcmp/s txcmp/s rxmcst/s
08:05:32 PM  lo  0.00  0.00  0.00  0.00  0.00  0.00  0.00
08:05:32 PM  eth0 1387.00 1469.00 652.12 256.98  0.00  0.00  0.00
	
Average:  IFACE rxpck/s txpck/s rxkB/s txkB/s rxcmp/s txcmp/s rxmcst/s
Average:   lo  0.00  0.00  0.00  0.00  0.00  0.00  0.00
Average:   eth0 1587.50 1696.00 791.29 296.29  0.00  0.00  0.00   
```
参数说明：  

- IFACE：LAN接口
- rxpck/s：每秒钟接收的数据包
- txpck/s：每秒钟发送的数据包
- rxbyt/s：每秒钟接收的字节数
- txbyt/s：每秒钟发送的字节数
- rxcmp/s：每秒钟接收的压缩数据包
- txcmp/s：每秒钟发送的压缩数据包
- rxmcst/s：每秒钟接收的多播数据包
- rxerr/s：每秒钟接收的坏数据包
- txerr/s：每秒钟发送的坏数据包
- coll/s：每秒冲突数
- rxdrop/s：因为缓冲充满，每秒钟丢弃的已接收数据包数
- txdrop/s：因为缓冲充满，每秒钟丢弃的已发送数据包数
- txcarr/s：发送数据包时，每秒载波错误数
- rxfram/s：每秒接收数据包的帧对齐错误数
- rxfifo/s：接收的数据包每秒FIFO过速的错误数
- txfifo/s：发送的数据包每秒FIFO过速的错误数  
	
**2. cat /proc/net/dev**   

Linux 内核提供了一种通过 /proc 文件系统，在运行时访问内核内部数据结构、改变内核设置的机制。proc文件系统是一个伪文件系统，它只存在内存当中，而不占用外存空间。它以文件系统的方式为访问系统内核数据的操作提供接口。用户和应用程序可以通过proc得到系统的信息，并可以改变内核的某些参数。由于系统的信息，如进程，是动态改变的，所以用户或应用程序读取proc文件时，proc文件系统是动态从系统内核读出所需信息并提交的。/proc文件系统中包含了很多目录，其中/proc/net/dev 保存了网络适配器及统计信息   

	[sre@CDVM-213017031 ~]$ cat /proc/net/dev
	Inter-| Receive | Transmit
	 face |bytes packets errs drop fifo frame compressed multicast|bytes packets errs drop fifo colls carrier compressed
	 lo:137052296 108029 0 0 0  0   0   0 137052296 108029 0 0 0  0  0   0
	 eth0:13661574714188 31346790620 0 0 0  0   0   0 5097461049535 27671144304 0 0    

最左边的表示接口的名字，Receive表示收包，Transmit表示发送包；  

- bytes表示收发的字节数
- packets表示收发正确的包量
- errs表示收发错误的包量
- drop表示收发丢弃的包量   

**实时监控脚本**  

```
#!/bin/bash

ethn=$1
while [[ true ]]; do   
  RX_pre=$(cat /proc/net/dev | grep $ethn | sed 's/:/ /g' | awk '{print $2}')      
  TX_pre=$(cat /proc/net/dev | grep $ethn | sed 's/:/ /g' | awk '{print $10}')   
  sleep 1
  RX_next=$(cat /proc/net/dev | grep $ethn | sed 's/:/ /g' | awk '{print $2}')
  TX_next=$(cat /proc/net/dev | grep $ethn | sed 's/:/ /g' | awk '{print $10}')

  clear
  echo -e "\t RX `date +%k:%M:%S` TX"

  RX=$((${RX_next}-${RX_pre}))
  TX=$((${TX_next}-${TX_pre}))

  if [[ $RX -lt 1024 ]];then
    RX="${RX}B/s"
  elif [[ $RX -gt 1048576 ]];then
    RX=$(echo $RX | awk '{print $1/1048576 "MB/s"}')
  else
    RX=$(echo $RX | awk '{print $1/1024 "KB/s"}')
  fi

  if [[ $TX -lt 1024 ]];then
    TX="${TX}B/s"
  elif [[ $TX -gt 1048576 ]];then
    TX=$(echo $TX | awk '{print $1/1048576 "MB/s"}')
  else
    TX=$(echo $TX | awk '{print $1/1024 "KB/s"}')
  fi

  echo -e "$ethn \t $RX   $TX "

done

```   
	
此脚本不需要额外再安装软件，并且可自定义欲查看接口，精确到小数，可根据流量大小灵活显示单位，默认采集间隔为1秒   

用法为：   

1、将脚本保存为可执行脚本文件，比如叫net.sh   

2、chmod +x ./net.sh 将文件改成可执行脚本   

3、sh net.sh eth0即可开始监看接口eth0流量，按ctrl+c退出   

**3.snmp 统计网络流量**   

以CISCO2900交换机为例做个简单上行流量计划的例子：   

1、获取CISCO2900端口1的上行总流量  

    snmpwalk -v 1 -c public 192.168.1.254 IF-MIB::ifInOctets.1
    返回结果
    IF-MIB::ifInOctets.1 = Counter32: 4861881  

2、五秒后再获取一次   
 
    snmpwalk -v 1 -c public 192.168.1.254 IF-MIB::ifInOctets.1
    返回结果
    IF-MIB::ifInOctets.1 = Counter32: 4870486  

3、计算结果  
	
	（后值48704863-前值4861881）/ 5＝1721b/s  （应该是BYTE） 