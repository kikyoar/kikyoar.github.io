---
layout:     post
title:      "No space left on device报错解决"
subtitle:   "fs.inotify.max_user_watches解析"
date:       2018-06-04
author:     "kikyoar"
header-img: "img/post-bg-linux-version.jpg"
tags:
    - Linux
---   

## 思路

**1.df -h 查看磁盘空间是否已满**

如果满了的话，去删除无用的大文件（比如日志）
   
**2.df -i 查看inodes空间是否已满**   

- 删除掉没用的临时文件，释放inodes  

	可以到/tmp目录下看看有没有很多sess_xxxx的session临时文件  

		ls -lt /tmp | wc -l  
	
	如果发现文件特别多，则：  
	
		find /tmp -type f -exec rm {} \;  

- 0字节的文件也会占用一个inode，也必须删除掉    
	遍历查找并删除
		
		find /home -type f -size 0 -exec rm {} \;  
		
- 遍历所有文件目录找出占空间大的文件，进行适当删除  

	先遍历出来占的空间大的目录，如果确定是某个目录下面，则/转换为该目录绝对路径   
	
		for i in /*; do echo $i; find $i | wc -l; done  

**3.lsop 命令查看被占用的进程文件**  

有些文件删除时还被其它进程占用，此时文件并未真正删除，只是标记为 deleted，只有进程结束后才会将文件真正从磁盘中清除

	# lsof | grep deleted
	mysqld 1952 2982 mysql 5u REG  254,1  0 127 /tmp/ibzMEe4z (deleted)
	mysqld 1952 2982 mysql 6u REG  254,1  0 146 /tmp/ibq6ZFge (deleted)
	mysqld 1952 2982 mysql 10u REG  254,1  0 150 /tmp/ibyNHH8y (deleted)
	apache2 2869  root 9u REG  254,1  0 168 /tmp/.ZendSem.2w14iv (deleted)
	apache2 2869  root 10w REG  0,16  0 11077 /run/lock/apache2/rewrite-map.2869 (deleted)
	...
	python 3102  root 1w REG  254,1 22412342132 264070 /var/log/nohup.out (deleted)  
	
原来是在后台运行的 Python 脚本，源源不断地将输出保存到 /var/log/nohup.out 文件中，文件大小居然达到了20G+！前阶段在后台运行脚本之后，就没再管过它。估计是我在 Python 运行过程中删掉了 nothup.out 文件，由于该文件被占用，所以只能先标记为 deleted，而未真正删除，最后导致磁盘爆满。重启进程

**4.You may have reached your quota of watches.**  


查看目前的最大值

	$cat /proc/sys/fs/inotify/max_user_watches
 
增加最大值

	$sudo sysctl fs.inotify.max_user_watches=16384
 
永久设置最大值

	$echo 16384 | sudo tee -a /proc/sys/fs/inotify/max_user_watches  
	


fs.inotify.max_queued_events：表示调用inotify_init时分配给inotify instance中可排队的event的数目的最大值，超出这个值的事件被丢弃，但会触发IN_Q_OVERFLOW事件  

fs.inotify.max_user_instances：表示每一个real user ID可创建的inotify instatnces的数量上限，默认128  

fs.inotify.max_user_watches：表示同一用户同时可以添加的watch数目（watch一般是针对目录，决定了同时同一用户可以监控的目录数量）  

**max_queued_events 是inotify管理的队列的最大长度，文件系统变化越频繁，这个值就应该越大。如果你在日志中看到Event Queue Overflow，说明max_queued_events太小需要调整参数后再次使用** 
		









   