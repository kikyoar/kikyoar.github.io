---
layout:     post
title:      "zabbix snmp 协议监控 dell iRDAC"
subtitle:   "zabbix监控Dell物理服务器"
date:       2018-05-22
author:     "kikyoar"
header-img: "img/post-bg-linux-version.jpg"
tags:
    - Zabbix
---  


**简单网络管理协议（SNMP），由一组网络管理的标准组成，包含一个应用层协议（application layer protocol）、数据库模型（database schema）和一组资源对象。该协议能够支持网络管理系统，用以监测连接到网络上的设备是否有任何引起管理上关注的情况。该协议是互联网工程工作小组 （IETF，Internet Engineering Task Force）定义的internet协议簇的一部分。SNMP的目标是管理互联网Internet上众多厂家生产的软硬件平台，因此SNMP受 Internet标准网络管理框架的影响也很大。SNMP已经出到第三个版本的协议，其功能较以前已经大大地加强和改进了**  

**iDRAC开启SNMP服务**   

登录之后单击 —> iDRAC设置 —> 网络 —> 服务 —> SNMP代理
登录之后单击 —> iDRAC设置 —> 网络 -> 启用LAN上IPMI  
*由于缺少IPMI模板，所以以下为snmp监控说明*  
zabbix 服务端通过snmp验证   


	[root@dev70 ~]# snmpget -v 2c -c <Community> <iDRAC IP> .1.3.6.1.4.1.674.10892.2.1.1.2.0
	[root@dev70 ~]# snmpget -v 2c -c xkeshi 192.168.184.200 .1.3.6.1.4.1.674.10892.2.1.1.2.0
	SNMPv2-SMI::enterprises.674.10892.2.1.1.2.0 = STRING: "iDRAC8"  

**配置zabbix**  

zabbix web界面 —> 管理 —> 一般 —> 值映射  
![zabbix值映射](http://kikyoar.com/img/zabbix_mapping.png)  
	
- DellDracDiskState  
	
			1 -> Unknown
			2 -> Ready
			3 -> Online
			4 -> Foreign
			5 -> Offline
			6 -> Blocked
			7 -> Failed
			8 -> Non-RAID
			9 -> Removed  
	
- Dell iDRAC Network Device Connection Status  
	
			1 -> Connected
			2 -> Disconnected  
	
- Dell Open Manage System Status  
	
			1 -> Other
			2 -> Unknown
			3 -> OK
			4 -> NonCritical
			5 -> Critical
			6 -> NonRecoverable  
	
- DellPowerState  
	
			1 -> Other
			2 -> Unknown
			3 -> Off
			4 -> On  
	
- Dell PSU State Settings  

			1 -> Unknown
			2 -> Online (state disabled)
			4 -> not Ready
			8 -> Fan Failure
			10 -> Online and Fan Failure
			16 -> On
			242 -> Online and OK  
	
- DellRaidLevel  

			1 -> Unknown
			2 -> RAID-0
			3 -> RAID-1
			4 -> RAID-5
			5 -> RAID-6
			6 -> RAID-10
			7 -> RAID-50
			8 -> RAID-60
			9 -> Concatenated RAID 1
			10 -> Concatenated RAID 5  
	
- DellRaidVolumeState  

			1 -> Unknown
			2 -> Online
			3 -> Failed
			4 -> Degraded		
			
- Dell_PSU_SensorState  

			1 -> Presence Detected
			2 -> PS Failure
			4 -> Predictuve Failure
			8 -> PS AC lost
			16 -> AC lost or out of range
			32 -> AC out of range but still present   

**配置全局变量{$SNMP_COMMUNITY_IDRAC}**  

zabbix web界面 —> 管理 —> 一般 —> 宏  
![zabbix全局宏](http://kikyoar.com/img/zabbix_Macro.png)   

**导入模板**    

[模板下载地址](https://github.com/endersonmaia/zabbix-templates/tree/master/dell/idrac)  
zabbix web界面 —> 配置 —> 模板 —> 导入  
Template_Dell_iDRAC_SNMPv2.zbx.xml    

**创建主机**    

zabbix web界面 —> 配置 —> 主机 —> 创建主机  
输入主机地址及SNMP地址和端口，保存  
