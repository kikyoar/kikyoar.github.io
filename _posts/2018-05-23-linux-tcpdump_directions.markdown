---
layout:     post
title:      "Tcpdump命令详解"
subtitle:   "学习如何使用Tcpdump"
date:       2018-05-23
author:     "kikyoar"
header-img: "img/post-bg-linux-version.jpg"
tags:
    - Linux
---   

**tcpdump是一个用于截取网络分组，并输出分组内容的工具，tcpdump可以将网络中传送的数据包的“头”完全截获下来提供分析。它支持针对网络层、协议、主机、网络或端口的过滤，并提供and、or、not等逻辑语句来帮助你去掉无用的信息**  



**tcpdump选项**   

tcpdump采用命令行方式，它的命令格式为：   

	tcpdump [ -AdDeflLnNOpqRStuUvxX ] [ -c count ]
	        [ -C file_size ] [ -F file ]
	        [ -i interface ] [ -m module ] [ -M secret ]
	        [ -r file ] [ -s snaplen ] [ -T type ] [ -w file ]
	        [ -W filecount ]
	        [ -E spi@ipaddr algo:secret,...  ]
	        [ -y datalinktype ] [ -Z user ]
	        [ expression ]  

****

**抓包选项：**  

	-c：指定要抓取的包数量。注意，是最终要获取这么多个包。例如，指定"-c 10"将获取10个包，但可能已经处理了100个包，只不过只有10个包是满足条件的包  
	
	-i interface：指定tcpdump需要监听的接口。若未指定该选项，将从系统接口列表中搜寻编号最小的已配置好的接口(不包括loopback接口，要抓取loopback接口使用tcpdump -i lo)，一旦找到第一个符合条件的接口，搜寻马上结束。可以使用'any'关键字表示所有网络接口  
	
	-n：对地址以数字方式显式，否则显式为主机名，也就是说-n选项不做主机名解析  
	
	-nn：除了-n的作用外，还把端口显示为数值，否则显示端口服务名  
	
	-N：不打印出host的域名部分。例如tcpdump将会打印'nic'而不是'nic.ddn.mil'  
	
	-P：指定要抓取的包是流入还是流出的包。可以给定的值为"in"、"out"和"inout"，默认为"inout"  
	
	-s len：设置tcpdump的数据包抓取长度为len，如果不设置默认将会是65535字节。对于要抓取的数据包较大时，长度设置不够可能会产生包截断，若出现包截断，输出行中会出现"[proto]"的标志(proto实际会显示为协议名)。但是抓取len越长，包的处理时间越长，并且会减少tcpdump可缓存的数据包的数量，从而会导致数据包的丢失，所以在能抓取我们想要的包的前提下，抓取长度越小越好    

****

**输出选项：**  

	-e：输出的每行中都将包括数据链路层头部信息，例如源MAC和目标MAC  
	
	-q：快速打印输出。即打印很少的协议相关信息，从而输出行都比较简短  
	
	-X：输出包的头部数据，会以16进制和ASCII两种方式同时输出  
	
	-XX：输出包的头部数据，会以16进制和ASCII两种方式同时输出，更详细  
	
	-v：当分析和打印的时候，产生详细的输出  
	
	-vv：产生比-v更详细的输出  
	
	-vvv：产生比-vv更详细的输出   
	
	-t 不在每一行中输出时间戳  
	
	-tt 在每一行中输出非格式化的时间戳  
	
	-ttt 输出本行和前面一行之间的时间差   
	
	-tttt 在每一行中输出由date处理的默认格式的时间戳  

****

**其他功能性选项：**  

	-D：列出可用于抓包的接口。将会列出接口的数值编号和接口名，它们都可以用于"-i"后  
	
	-F：从文件中读取抓包的表达式。若使用该选项，则命令行中给定的其他表达式都将失效  
	
	-w：将抓包数据输出到文件中而不是标准输出。可以同时配合"-G time"选项使得输出文件每time秒就自动切换到另一个文件。可通过"-r"选项载入这些文件以进行分析和打印  
	
	-r：从给定的数据包文件中读取数据。使用"-"表示从标准输入中读取   

****

**tcpdump的表达式**   

表达式是一个正则表达式，tcpdump利用它作为过滤报文的条件，如果一个报文满足表达式的条件，则这个报文将会被捕获。如果没有给出任何条件，则网络上所有的信息包将会被截获。在表达式中一般如下几种类型的关键字：

第一种是关于类型的关键字，主要包括host，net，port, 例如 host 210.27.48.2，指明 210.27.48.2是一台主机，net 202.0.0.0 指明 202.0.0.0是一个网络地址，port 23 指明端口号是23。如果没有指定类型，缺省的类型是host.

第二种是确定传输方向的关键字，主要包括src , dst ,dst or src, dst and src ,这些关键字指明了传输的方向。举例说明，src 210.27.48.2 ,指明ip包中源地址是210.27.48.2 , dst net 202.0.0.0 指明目的网络地址是202.0.0.0 。如果没有指明方向关键字，则缺省是src or dst关键字。

第三种是协议的关键字，主要包括fddi,ip,arp,rarp,tcp,udp等类型。Fddi指明是在FDDI(分布式光纤数据接口网络)上的特定的网络协议，实际上它是”ether”的别名，fddi和ether具有类似的源地址和目的地址，所以可以将fddi协议包当作ether的包进行处理和分析。其他的几个关键字就是指明了监听的包的协议内容。如果没有指定任何协议，则tcpdump将会监听所有协议的信息包。

除了这三种类型的关键字之外，其他重要的关键字如下：gateway, broadcast,less,greater,还有三种逻辑运算，取非运算是 ‘not ‘ ‘! ‘, 与运算是’and’,'&&’;或运算 是’or’ ,’││’；这些关键字可以组合起来构成强大的组合条件来满足人们的需要。   


**tcpdump示例**    

- 监视指定网络接口的数据包   

		tcpdump -i eth1  

	如果不指定网卡，默认tcpdump只会监视第一个网络接口，如eth0   

- 监视指定主机的数据包，例如所有进入或离开jumporange的数据包   

		tcpdump host jumporange   

- 打印主机helios<-->hot或helios<-->ace之间通信的数据包   

		tcpdump host helios and \( hot or ace \)   
		
- 打印ace与任何其他主机之间通信的IP数据包,但不包括与helios之间的数据包  

		tcpdump ip host ace and not helios  

- 截获主机hostname发送的所有数据  

		tcpdump src host hostname  

- 监视所有发送到主机hostname的数据包  

		tcpdump dst host hostname  

- 监视指定主机和端口的数据包  

		tcpdump tcp port 22 and host hostname  

- 对本机的udp 123端口进行监视(123为ntp的服务端口)  

		tcpdump udp port 123  

- 监视指定网络的数据包，如本机与192.168网段通信的数据包，"-c 10"表示只抓取10个包  

		tcpdump -c 10 net 192.168  
- 打印所有通过网关snup的ftp数据包(注意,表达式被单引号括起来了,这可以防止shell对其中的括号进行错误解析)  

		tcpdump 'gateway snup and (port ftp or ftp-data)'    

- 抓取ping包  

		[root@server2 ~]# tcpdump -c 5 -nn -i eth0 icmp 
		
		tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
		listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
		12:11:23.273638 IP 192.168.100.70 > 192.168.100.62: ICMP echo request, id 16422, seq 10, length 64
		12:11:23.273666 IP 192.168.100.62 > 192.168.100.70: ICMP echo reply, id 16422, seq 10, length 64
		12:11:24.356915 IP 192.168.100.70 > 192.168.100.62: ICMP echo request, id 16422, seq 11, length 64
		12:11:24.356936 IP 192.168.100.62 > 192.168.100.70: ICMP echo reply, id 16422, seq 11, length 64
		12:11:25.440887 IP 192.168.100.70 > 192.168.100.62: ICMP echo request, id 16422, seq 12, length 64
		5 packets captured
		6 packets received by filter
		0 packets dropped by kernel  
		
		
	如果明确要抓取主机为192.168.100.70对本机的ping，则使用and操作符   


		[root@server2 ~]# tcpdump -c 5 -nn -i eth0 icmp and src 192.168.100.62
		
		tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
		listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
		12:09:29.957132 IP 192.168.100.70 > 192.168.100.62: ICMP echo request, id 16166, seq 1, length 64
		12:09:31.041035 IP 192.168.100.70 > 192.168.100.62: ICMP echo request, id 16166, seq 2, length 64
		12:09:32.124562 IP 192.168.100.70 > 192.168.100.62: ICMP echo request, id 16166, seq 3, length 64
		12:09:33.208514 IP 192.168.100.70 > 192.168.100.62: ICMP echo request, id 16166, seq 4, length 64
		12:09:34.292222 IP 192.168.100.70 > 192.168.100.62: ICMP echo request, id 16166, seq 5, length 64
		5 packets captured
		5 packets received by filter
		0 packets dropped by kernel  
		
	注意不能直接写icmp src 192.168.100.70，因为icmp协议不支持直接应用host这个type   

- 抓取到本机22端口包  

		[root@server2 ~]# tcpdump -c 10 -nn -i eth0 tcp dst port 22  
		
		tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
		listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
		12:06:57.574293 IP 192.168.100.1.5788 > 192.168.100.62.22: Flags [.], ack 535528834, win 2053, length 0
		12:06:57.629125 IP 192.168.100.1.5788 > 192.168.100.62.22: Flags [.], ack 193, win 2052, length 0
		12:06:57.684688 IP 192.168.100.1.5788 > 192.168.100.62.22: Flags [.], ack 385, win 2051, length 0
		12:06:57.738977 IP 192.168.100.1.5788 > 192.168.100.62.22: Flags [.], ack 577, win 2050, length 0
		12:06:57.794305 IP 192.168.100.1.5788 > 192.168.100.62.22: Flags [.], ack 769, win 2050, length 0
		12:06:57.848720 IP 192.168.100.1.5788 > 192.168.100.62.22: Flags [.], ack 961, win 2049, length 0
		12:06:57.904057 IP 192.168.100.1.5788 > 192.168.100.62.22: Flags [.], ack 1153, win 2048, length 0
		12:06:57.958477 IP 192.168.100.1.5788 > 192.168.100.62.22: Flags [.], ack 1345, win 2047, length 0
		12:06:58.014338 IP 192.168.100.1.5788 > 192.168.100.62.22: Flags [.], ack 1537, win 2053, length 0
		12:06:58.069361 IP 192.168.100.1.5788 > 192.168.100.62.22: Flags [.], ack 1729, win 2052, length 0
		10 packets captured
		10 packets received by filter
		0 packets dropped by kernel  
		
- 解析包数据  

		[root@server2 ~]# tcpdump -c 2 -q -XX -vvv -nn -i eth0 tcp dst port 22
		tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
		12:15:54.788812 IP (tos 0x0, ttl 64, id 19303, offset 0, flags [DF], proto TCP (6), length 40)
		    192.168.100.1.5788 > 192.168.100.62.22: tcp 0
		        0x0000:  000c 2908 9234 0050 56c0 0008 0800 4500  ..)..4.PV.....E.
		        0x0010:  0028 4b67 4000 4006 a5d8 c0a8 6401 c0a8  .(Kg@.@.....d...
		        0x0020:  643e 169c 0016 2426 5fd6 1fec 2b62 5010  d>....$&_...+bP.
		        0x0030:  0803 7844 0000 0000 0000 0000            ..xD........
		12:15:54.842641 IP (tos 0x0, ttl 64, id 19304, offset 0, flags [DF], proto TCP (6), length 40)
		    192.168.100.1.5788 > 192.168.100.62.22: tcp 0
		        0x0000:  000c 2908 9234 0050 56c0 0008 0800 4500  ..)..4.PV.....E.
		        0x0010:  0028 4b68 4000 4006 a5d7 c0a8 6401 c0a8  .(Kh@.@.....d...
		        0x0020:  643e 169c 0016 2426 5fd6 1fec 2d62 5010  d>....$&_...-bP.
		        0x0030:  0801 7646 0000 0000 0000 0000            ..vF........
		2 packets captured
		2 packets received by filter
		0 packets dropped by kernel  



