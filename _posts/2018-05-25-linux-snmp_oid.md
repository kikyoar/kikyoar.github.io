---
layout:     post
title:      "SNMP获取交换机端口流量分析及OID分析"
subtitle:   "SNMP使用方法"
date:       2018-05-25
author:     "kikyoar"
header-img: "img/post-bg-linux-version.jpg"
tags:
    - Linux
---   

**一般端口流量分析针对普通网络设备的端口，MIB的相关定义是Interface组，主要管理如下信息:**  

- ifIndex 端口索引号  
- ifDescr 端口描述
- ifType 端口类型
- ifMtu 最大传输包字节数
- ifSpeed 端口速度
- ifPhysAddress 物理地址
- ifOperStatus 操作状态
- ifLastChange 上次状态更新时间
- *ifInOctets 输入字节数
- *ifInUcastPkts 输入非广播包数
- *ifInNUcastPkts 输入广播包数
- *ifInDiscards 输入包丢弃数
- *ifInErrors 输入包错误数
- *ifInUnknownProtos 输入未知协议包数
- *ifOutOctets 输出字节数
- *ifOutUcastPkts 输出非广播包数
- *ifOutNUcastPkts 输出广播包数
- *ifOutDiscards 输出包丢弃数
- *ifOutErrors 输出包错误数
- ifOutQLen 输出队长  

**其中，\*号标识的是与网络流量有关的信息**  

举例说明：  

1.查看所有端口描述信息（可查看端口与所对应的ID号）

	[root@redcock lifeng]# snmpwalk -v 2c -c lifeng 192.168.100.100   ifDescr  
	
2.查看所有端口进入流量字节数  

	[root@redcock lifeng]# snmpwalk -v 2c -c lifeng 192.168.100.100   ifInOctets  

3.查看ID号是57的端口进入流量字节数（ID号不一定与端口号对应）  

	[root@redcock lifeng]# snmpwalk -v 2c -c lifeng 192.168.100.100   ifInOctets.57    
	
	IF-MIB::ifInOctets.57 = Counter32: 3877397126  

****  

收集整理一些Linux下snmp常用的OID，用做服务器监控很不错  

**服务器负载：**  

	1 minute Load: .1.3.6.1.4.1.2021.10.1.3.1
	
	5 minute Load: .1.3.6.1.4.1.2021.10.1.3.2
	
	15 minute Load: .1.3.6.1.4.1.2021.10.1.3.3

**CPU信息：**  

	percentage of user CPU time: .1.3.6.1.4.1.2021.11.9.0
	
	raw user cpu time: .1.3.6.1.4.1.2021.11.50.0
	
	percentages of system CPU time: .1.3.6.1.4.1.2021.11.10.0
	
	raw system cpu time: .1.3.6.1.4.1.2021.11.52.0
	
	percentages of idle CPU time: .1.3.6.1.4.1.2021.11.11.0
	
	raw idle cpu time: .1.3.6.1.4.1.2021.11.53.0
	
	raw nice cpu time: .1.3.6.1.4.1.2021.11.51.0  
	
**内存使用：**   

	Total Swap Size: .1.3.6.1.4.1.2021.4.3.0
	
	Available Swap Space: .1.3.6.1.4.1.2021.4.4.0
	
	Total RAM in machine: .1.3.6.1.4.1.2021.4.5.0
	
	Total RAM used: .1.3.6.1.4.1.2021.4.6.0
	
	Total RAM Free: .1.3.6.1.4.1.2021.4.11.0
	
	Total RAM Shared: .1.3.6.1.4.1.2021.4.13.0
	
	Total RAM Buffered: .1.3.6.1.4.1.2021.4.14.0
	
	Total Cached Memory: .1.3.6.1.4.1.2021.4.15.0  
	
**硬盘使用：**   

	Path where the disk is mounted: .1.3.6.1.4.1.2021.9.1.2.1
	
	Path of the device for the partition: .1.3.6.1.4.1.2021.9.1.3.1
	
	Total size of the disk/partion(kBytes): .1.3.6.1.4.1.2021.9.1.6.1
	
	Available space on the disk: .1.3.6.1.4.1.2021.9.1.7.1
	
	Used space on the disk: .1.3.6.1.4.1.2021.9.1.8.1
	
	Percentage of space used on disk: .1.3.6.1.4.1.2021.9.1.9.1
	
	Percentage of inodes used on disk: .1.3.6.1.4.1.2021.9.1.10.1  
	
**系统运行时间：**   

	1.3.6.1.2.1.1.3.0   
	
**SNMP System Group：**    

	sysDescr 1.3.6.1.2.1.1.1
	
	sysObjectID 1.3.6.1.2.1.1.2
	
	sysUpTime 1.3.6.1.2.1.1.3
	
	sysContact 1.3.6.1.2.1.1.4
	
	sysName 1.3.6.1.2.1.1.5
	
	sysLocation 1.3.6.1.2.1.1.6
	
	sysServices 1.3.6.1.2.1.1.7   

**SNMP Interfaces Group：**   

	ifNumber 1.3.6.1.2.1.2.1
	
	ifTable 1.3.6.1.2.1.2.2
	
	ifEntry 1.3.6.1.2.1.2.2.1
	
	ifIndex 1.3.6.1.2.1.2.2.1.1
	
	ifDescr 1.3.6.1.2.1.2.2.1.2
	
	ifType 1.3.6.1.2.1.2.2.1.3
	
	ifMtu 1.3.6.1.2.1.2.2.1.4
	
	ifSpeed 1.3.6.1.2.1.2.2.1.5
	
	ifPhysAddress 1.3.6.1.2.1.2.2.1.6
	
	ifAdminStatus 1.3.6.1.2.1.2.2.1.7
	
	ifOperStatus 1.3.6.1.2.1.2.2.1.8
	
	ifLastChange 1.3.6.1.2.1.2.2.1.9
	
	ifInOctets 1.3.6.1.2.1.2.2.1.10
	
	ifInUcastPkts 1.3.6.1.2.1.2.2.1.11
	
	ifInNUcastPkts 1.3.6.1.2.1.2.2.1.12
	
	ifInDiscards 1.3.6.1.2.1.2.2.1.13
	
	ifInErrors 1.3.6.1.2.1.2.2.1.14
	
	ifInUnknownProtos 1.3.6.1.2.1.2.2.1.15
	
	ifOutOctets 1.3.6.1.2.1.2.2.1.16
	
	ifOutUcastPkts 1.3.6.1.2.1.2.2.1.17
	
	ifOutNUcastPkts 1.3.6.1.2.1.2.2.1.18
	
	ifOutDiscards 1.3.6.1.2.1.2.2.1.19
	
	ifOutErrors 1.3.6.1.2.1.2.2.1.20
	
	ifOutQLen 1.3.6.1.2.1.2.2.1.21
	
	ifSpecific 1.3.6.1.2.1.2.2.1.22  
	
**SNMP IP Group:**   

	ipForwarding 1.3.6.1.2.1.4.1
	
	ipDefaultTTL 1.3.6.1.2.1.4.2
	
	ipInReceives 1.3.6.1.2.1.4.3
	
	ipInHdrErrors 1.3.6.1.2.1.4.4
	
	ipInAddrErrors 1.3.6.1.2.1.4.5
	
	ipForwDatagrams 1.3.6.1.2.1.4.6
	
	ipInUnknownProtos 1.3.6.1.2.1.4.7
	
	ipInDiscards 1.3.6.1.2.1.4.8
	
	ipInDelivers 1.3.6.1.2.1.4.9
	
	ipOutRequests 1.3.6.1.2.1.4.10
	
	ipOutDiscards 1.3.6.1.2.1.4.11
	
	ipOutNoRoutes 1.3.6.1.2.1.4.12
	
	ipReasmTimeout 1.3.6.1.2.1.4.13
	
	ipReasmReqds 1.3.6.1.2.1.4.14
	
	ipReasmOKs 1.3.6.1.2.1.4.15
	
	ipReasmFails 1.3.6.1.2.1.4.16
	
	ipFragsOKs 1.3.6.1.2.1.4.17
	
	ipFragsFails 1.3.6.1.2.1.4.18
	
	ipFragCreates 1.3.6.1.2.1.4.19
	
	ipAddrTable 1.3.6.1.2.1.4.20
	
	ipAddrEntry 1.3.6.1.2.1.4.20.1
	
	ipAdEntAddr 1.3.6.1.2.1.4.20.1.1
	
	ipAdEntIfIndex 1.3.6.1.2.1.4.20.1.2
	
	ipAdEntNetMask 1.3.6.1.2.1.4.20.1.3
	
	ipAdEntBcastAddr 1.3.6.1.2.1.4.20.1.4
	
	ipAdEntReasmMaxSize 1.3.6.1.2.1.4.20.1.5  
	
**SNMP ICMP Group:**   

	icmpInMsgs 1.3.6.1.2.1.5.1
	
	icmpInErrors 1.3.6.1.2.1.5.2
	
	icmpInDestUnreachs 1.3.6.1.2.1.5.3
	
	icmpInTimeExcds 1.3.6.1.2.1.5.4
	
	icmpInParmProbs 1.3.6.1.2.1.5.5
	
	icmpInSrcQuenchs 1.3.6.1.2.1.5.6
	
	icmpInRedirects 1.3.6.1.2.1.5.7
	
	icmpInEchos 1.3.6.1.2.1.5.8
	
	icmpInEchoReps 1.3.6.1.2.1.5.9
	
	icmpInTimestamps 1.3.6.1.2.1.5.10
	
	icmpInTimestampReps 1.3.6.1.2.1.5.11
	
	icmpInAddrMasks 1.3.6.1.2.1.5.12
	
	icmpInAddrMaskReps 1.3.6.1.2.1.5.13
	
	icmpOutMsgs 1.3.6.1.2.1.5.14
	
	icmpOutErrors 1.3.6.1.2.1.5.15
	
	icmpOutDestUnreachs 1.3.6.1.2.1.5.16
	
	icmpOutTimeExcds 1.3.6.1.2.1.5.17
	
	icmpOutParmProbs 1.3.6.1.2.1.5.18
	
	icmpOutSrcQuenchs 1.3.6.1.2.1.5.19
	
	icmpOutRedirects 1.3.6.1.2.1.5.20
	
	icmpOutEchos 1.3.6.1.2.1.5.21
	
	icmpOutEchoReps 1.3.6.1.2.1.5.22
	
	icmpOutTimestamps 1.3.6.1.2.1.5.23
	
	icmpOutTimestampReps 1.3.6.1.2.1.5.24
	
	icmpOutAddrMasks 1.3.6.1.2.1.5.25
	
	icmpOutAddrMaskReps 1.3.6.1.2.1.5.26   

**SNMP TCP Group:**  

	tcpRtoAlgorithm 1.3.6.1.2.1.6.1
	
	tcpRtoMin 1.3.6.1.2.1.6.2
	
	tcpRtoMax 1.3.6.1.2.1.6.3
	
	tcpMaxConn 1.3.6.1.2.1.6.4
	
	tcpActiveOpens 1.3.6.1.2.1.6.5
	
	tcpPassiveOpens 1.3.6.1.2.1.6.6
	
	tcpAttemptFails 1.3.6.1.2.1.6.7
	
	tcpEstabResets 1.3.6.1.2.1.6.8
	
	tcpCurrEstab 1.3.6.1.2.1.6.9
	
	tcpInSegs 1.3.6.1.2.1.6.10
	
	tcpOutSegs 1.3.6.1.2.1.6.11
	
	tcpRetransSegs 1.3.6.1.2.1.6.12
	
	tcpConnTable 1.3.6.1.2.1.6.13
	
	tcpConnEntry 1.3.6.1.2.1.6.13.1
	
	tcpConnState 1.3.6.1.2.1.6.13.1.1
	
	tcpConnLocalAddress 1.3.6.1.2.1.6.13.1.2
	
	tcpConnLocalPort 1.3.6.1.2.1.6.13.1.3
	
	tcpConnRemAddress 1.3.6.1.2.1.6.13.1.4
	
	tcpConnRemPort 1.3.6.1.2.1.6.13.1.5
	
	tcpInErrs 1.3.6.1.2.1.6.14
	
	tcpOutRsts 1.3.6.1.2.1.6.15   

**SNMP UDP Group:**  

	udpInDatagrams 1.3.6.1.2.1.7.1
	
	udpNoPorts 1.3.6.1.2.1.7.2
	
	udpInErrors 1.3.6.1.2.1.7.3
	
	udpOutDatagrams 1.3.6.1.2.1.7.4
	
	udpTable 1.3.6.1.2.1.7.5
	
	udpEntry 1.3.6.1.2.1.7.5.1
	
	udpLocalAddress 1.3.6.1.2.1.7.5.1.1
	
	udpLocalPort 1.3.6.1.2.1.7.5.1.2   

**SNMP Group:**  

	snmpInPkts 1.3.6.1.2.1.11.1
	
	snmpOutPkts 1.3.6.1.2.1.11.2
	
	snmpInBadVersions 1.3.6.1.2.1.11.3
	
	snmpInBadCommunityNames 1.3.6.1.2.1.11.4
	
	snmpInBadCommunityUses 1.3.6.1.2.1.11.5
	
	snmpInASNParseErrs 1.3.6.1.2.1.11.6
	
	NOT USED 1.3.6.1.2.1.11.7
	
	snmpInTooBigs 1.3.6.1.2.1.11.8
	
	snmpInNoSuchNames 1.3.6.1.2.1.11.9
	
	snmpInBadValues 1.3.6.1.2.1.11.10
	
	snmpInReadOnlys 1.3.6.1.2.1.11.11
	
	snmpInGenErrs 1.3.6.1.2.1.11.12
	
	snmpInTotalReqVars 1.3.6.1.2.1.11.13
	
	snmpInTotalSetVars 1.3.6.1.2.1.11.14
	
	snmpInGetRequests 1.3.6.1.2.1.11.15
	
	snmpInGetNexts 1.3.6.1.2.1.11.16
	
	snmpInSetRequests 1.3.6.1.2.1.11.17
	
	snmpInGetResponses 1.3.6.1.2.1.11.18
	
	snmpInTraps 1.3.6.1.2.1.11.19
	
	snmpOutTooBigs 1.3.6.1.2.1.11.20
	
	snmpOutNoSuchNames 1.3.6.1.2.1.11.21
	
	snmpOutBadValues 1.3.6.1.2.1.11.22
	
	NOT USED 1.3.6.1.2.1.11.23
	
	snmpOutGenErrs 1.3.6.1.2.1.11.24
	
	snmpOutGetRequests 1.3.6.1.2.1.11.25
	
	snmpOutGetNexts 1.3.6.1.2.1.11.26
	
	snmpOutSetRequests 1.3.6.1.2.1.11.27
	
	snmpOutGetResponses 1.3.6.1.2.1.11.28
	
	snmpOutTraps 1.3.6.1.2.1.11.29
	
	snmpEnableAuthenTraps 1.3.6.1.2.1.11.30  
	
**实例**  
如果知道交换机端口查找1口的进入流量方法：  

1.先查看交换机端口描述

	[root@gsunicom-web zabbix]# snmpwalk -v 2c -c hadoop 192.168.100.1 ifDescr
	IF-MIB::ifDescr.1 = STRING: GigabitEthernet0/0
	IF-MIB::ifDescr.2 = STRING: Null0
	IF-MIB::ifDescr.3 = STRING: GigabitEthernet1/0/1
	IF-MIB::ifDescr.4 = STRING: GigabitEthernet1/0/2
	IF-MIB::ifDescr.5 = STRING: GigabitEthernet1/0/3
	IF-MIB::ifDescr.6 = STRING: GigabitEthernet1/0/4
	IF-MIB::ifDescr.7 = STRING: GigabitEthernet1/0/5
	IF-MIB::ifDescr.8 = STRING: GigabitEthernet1/0/6   

2.查看1口的进入流量
根据描述“IF-MIB::ifDescr.3”，ifInOctetsOID号为 1.3.6.1.2.1.2.2.1.10  

	[root@gsunicom-web zabbix]# snmpwalk -v 2c -c hadoop 192.168.100.1 1.3.6.1.2.1.2.2.1.10.3
	IF-MIB::ifInOctets.3 = Counter32: 3414918307
	
3.以下为换算公式  
![linux_traffic_conversion](http://kikyoar.com/img/traffic_conversion.png) 

