---
layout:     post
title:      "Linux lsof命令详解"
subtitle:   "Linux下删除文件恢复"
date:       2019-05-06
author:     "kikyoar"
header-img: "img/post-bg-linux-version.jpg"
tags:
    - Linux
--- 

lsof命令用于查看你进程开打的文件，打开文件的进程，进程打开的端口(TCP、UDP)。找回/恢复删除的文件。是十分方便的系统监视工具，因为lsof命令需要访问核心内存和各种文件，所以需要root用户执行  

在linux环境下，任何事物都以文件的形式存在，通过文件不仅仅可以访问常规数据，还可以访问网络连接和硬件。所以如传输控制协议 (TCP) 和用户数据报协议 (UDP) 套接字等，系统在后台都为该应用程序分配了一个文件描述符，无论这个文件的本质如何，该文件描述符为应用程序与基础操作系统之间的交互提供了通用接口。因为应用程序打开文件的描述符列表提供了大量关于这个应用程序本身的信息，因此通过lsof工具能够查看这个列表对系统监测以及排错将是很有帮助的  


### 输出信息含义

在终端下输入lsof即可显示系统打开的文件，因为 lsof 需要访问核心内存和各种文件，所以必须以**root 用户的身份运行它才能够充分地发挥其功能**  

直接输入lsof部分输出为:  

	COMMAND     PID        USER   FD      TYPE             DEVICE SIZE/OFF       NODE NAME
	init          1        root  cwd       DIR                8,1     4096          2 /
	init          1        root  rtd       DIR                8,1     4096          2 /
	init          1        root  txt       REG                8,1   150584     654127 /sbin/init
	udevd       415        root    0u      CHR                1,3      0t0       6254 /dev/null
	udevd       415        root    1u      CHR                1,3      0t0       6254 /dev/null
	udevd       415        root    2u      CHR                1,3      0t0       6254 /dev/null
	udevd       690        root  mem       REG                8,1    51736     302589 /lib/x86_64-linux-gnu/libnss_files-2.13.so
	syslogd    1246      syslog    2w      REG                8,1    10187     245418 /var/log/auth.log
	syslogd    1246      syslog    3w      REG                8,1    10118     245342 /var/log/syslog
	dd         1271        root    0r      REG                0,3        0 4026532038 /proc/kmsg
	dd         1271        root    1w     FIFO               0,15      0t0        409 /run/klogd/kmsg
	dd         1271        root    2u      CHR                1,3      0t0       6254 /dev/null  
	
每行显示一个打开的文件，若不指定条件默认将显示所有进程打开的所有文件  

lsof输出各列信息的意义如下：

- COMMAND：		进程的名称 PID：进程标识符
- USER：       	进程所有者
- FD：           文件描述符，应用程序通过文件描述符识别该文件。如cwd、txt等
- TYPE：        	文件类型，如DIR、REG等
- DEVICE：    	指定磁盘的名称
- SIZE：        	文件的大小
- NODE：      	索引节点（文件在磁盘上的标识）
- NAME：       	打开文件的确切名称

FD列中的文件描述符cwd值表示应用程序的当前工作目录，这是该应用程序启动的目录，除非它本身对这个目录进行更改,txt类型的文件是程序代码，如应用程序二进制文件本身或共享库，如上列表中显示的/sbin/init 程序。其次数值表示应用程序的文件描述符，这是打开该文件时返回的一个整数。如上的最后一行文件/dev/initctl，其文件描述符为 10。u 表示该文件被打开并处于读取/写入模式，而不是只读 （r）或只写 (w) 模式。同时还有大写的W 表示该应用程序具有对整个文件的写锁。该文件描述符用于确保每次只能打开一个应用程序实例。初始打开每个应用程序时，都具有三个文件描述符，从 0 到 2，分别表示标准输入、输出和错误流。所以大多数应用程序所打开的文件的 FD 都是从 3 开始。

Type 列则比较直观，**文件和目录分别称为 REG 和 DIR。而CHR 和 BLK，分别表示字符和块设备；或者 UNIX、FIFO 和 IPv4，分别表示 UNIX 域套接字、先进先出 (FIFO) 队列和网际协议 (IP) 套接字**

### 常用参数  

lsof语法格式是：
lsof ［options］ filename

	lsof abc.txt 			显示开启文件abc.txt的进程
	lsof -c abc 			显示abc进程现在打开的文件
	lsof -p 30297 			显示哪些文件被pid为30297的进程打开
	lsof -c -p 1234 		列出进程号为1234的进程所打开的文件
	lsof -g gid 			显示归属gid的进程情况
	lsof +d /usr/local/ 	显示目录下被进程开启的文件
	lsof +D /usr/local/ 	同上，但是会搜索目录下的目录，时间较长
	lsof -d 4 				显示使用fd为4的进程
	lsof -u 1000 			查看uid是100的用户的进程的文件使用情况
	lsof -u tony 			查看用户tony的进程的文件使用情况
	lsof -u ^tony			查看不是用户tony的进程的文件使用情况(^是取反的意思)
	lsof -i					显示所有打开的端口
	lsof -i:80				显示打开80端口的进程
	lsof -i -U				显示所有打开的端口和UNIX domain文件
	
	
	lsof -i[46] [protocol][@hostname|hostaddr][:service|port]
	--> IPv4 or IPv6
	  protocol --> TCP or UDP
	  hostname --> Internet host name
	  hostaddr --> IPv4地址
	  service --> /etc/service中的 service name (可以不止一个)
	  port --> 端口号 (可以不止一个)
	lsof -i@192.168.1.101  	显示192.168.1.101主机的连接
	lsof -i@192.168.1.130:51461 	显示来自于192.168.1.130的51461端口的连接
	lsof -i -sTCP:LISTEN  		找出TCP的监听端口
	lsof -i -sTCP:ESTABLISHED  	找出已建立连接的连接。
	lsof -i  UDP:8668 	列出udp8668端口的连接
	lsof -i @www.baidu.com 		显示哪个进程跟www.baidu.com在连接

### lsof使用实例

- 查找谁在使用文件系统  
	在卸载文件系统时，如果该文件系统中有任何打开的文件，操作通常将会失败。那么通过lsof可以找出那些进程在使用当前要卸载的文件系统，如下：

		# lsof /GTES11/ 
		COMMAND PID USER FD TYPE DEVICE SIZE NODE NAME 
		bash 4208 root cwd DIR 3,1 4096 2 /GTES11/ 
		vim 4230 root cwd DIR 3,1 4096 2 /GTES11/

	在这个示例中，用户root正在其/GTES11目录中进行一些操作。一个 bash是实例正在运行，并且它当前的目录为/GTES11，另一个则显示的是vim正在编辑/GTES11下的文件。要成功地卸载/GTES11，应该在通知用户以确保情况正常之后，中止这些进程。 这个示例说明了应用程序的当前工作目录非常重要，因为它仍保持着文件资源，并且可以防止文件系统被卸载。这就是为什么大部分守护进程（后台进程）将它们的目录更改为根目录、或服务特定的目录（如 sendmail 示例中的 /var/spool/mqueue）的原因，以避免该守护进程阻止卸载不相关的文件系统  


- 误删除文件进程还在的情况可以用lsof:  
	这种一般是有活动的进程存在持续标准输入或输出，到时文件被删除后，进程PID依旧存在。这也是有些服务器删除一些文件但是磁盘不释放的原因  
	
	打开一个终端对一个测试文件做cat追加操作：  
	
		[root@docking ~]# echo "This is DeleteFile test." > deletefile.txt
		[root@docking ~]# ls
		deletefile.txt
		[root@docking ~]# cat >> deletefile.txt 
		Add SomeLine into deletefile for fun.  
		
	此时，删除文件rm -f deletefile.txt
	
		[root@docking ~]# rm -f deletefile.txt 
		[root@docking ~]# ls
	
	命令查看这个目录，文件已经不存在了，那么现在我们将其恢复出来  
	
	- lsof查看删除的文件进程是否还存在。
	- 如没有安装请自行yum install lsof或者apt-get install lsof  
		- 类似这种情况，我们可以先lsof查看删除的文件 是否还在

				[root@docking ~]# lsof | grep deletefile
				cat       21796          root    1w      REG              253,1        63     138860 /root/deletefile.txt (deleted)

		- 恢复cp /proc/pid/fd/1 /指定目录/文件名  
			进入进程目录，一般是进入/proc/pid/fd/,针对当前情况： 
		
				[root@docking ~]# cd /proc/21796/fd
				[root@docking fd]# ll
				总用量 0
				lrwx------ 1 root root 64 1月  18 22:21 0 -> /dev/pts/0
				l-wx------ 1 root root 64 1月  18 22:21 1 -> /root/deletefile.txt (deleted)
				lrwx------ 1 root root 64 1月  18 22:21 2 -> /dev/pts/0  
			
		- 恢复操作：  

				[root@docking fd]# cp 1 ~/deletefile.txt.backup
				[root@docking fd]# cat ~/deletefile.txt.backup 
				This is DeleteFile test.
				Add SomeLine into deletefile for fun.

		- 误删除的文件进程已经不存在，借助于工具还原，此处由于概率恢复极低，不在赘述

	

