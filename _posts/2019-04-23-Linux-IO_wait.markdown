---
layout:     post
title:      "发现并解决linux高I/O Wait问题"
subtitle:   "如何发现linux中引起高io等待的进程"
date:       2019-04-23
author:     "kikyoar"
header-img: "img/post-bg-linux-version.jpg"
tags:
    - Linux
---  

[部分节选自酷喃](http://coolnull.com/4444.html)

linux用很多可用的工具可以用来发现排错，有些很容易使用，有些用法则比较高级

查看I/O wait问题不仅需要使用一些高级工具，也需要一些基本工具的高级用法。I/O wait之所以难以排查是因为默认有太多的工具告诉你系统I\/O阻塞，但没那么多工具可以帮你缩小范围以便确定出是哪个或哪些进程引起的问题   

- 首先回答是不是I/O引起系统缓慢  
确定是不是I/O引起系统缓慢，你可以使用很多工具但最简单的还是unix命令**top**   
  
  ![top1](http://kikyoar.com/img/top1.jpg)
  
  
  从CPU(s) 这行你可以看出当前CPU I/O Wait的情况；越高的wa表示越多的cpu资源在等待I/O  
  
		wa -- iowait
		Amount of time the CPU has been waiting for I/O to complete.    //cpu已经等待I/O完成的时间   
		
- 查找哪个硬盘正在被写入  
  上面的top命令从系统面大体展示了I/O Wait，但它没有告诉你哪个硬盘正在被影响；为此我们需要使用iostat命令  

		[root@coolnull ~]# yum install sysstat
		[root@coolnull ~]# iostat -x 2 5
		 avg-cpu: %user %nice %system %iowait %steal %idle
		  3.66 0.00 47.64 48.69 0.00 0.00
		
		 Device: rrqm/s wrqm/s r/s w/s rkB/s wkB/s avgrq-sz avgqu-sz await r_await w_await svctm %util
		 sda 44.50 39.27 117.28 29.32 11220.94 13126.70 332.17 65.77 462.79 9.80 2274.71 7.60 111.41
		 dm-0 0.00 0.00 83.25 9.95 10515.18 4295.29 317.84 57.01 648.54 16.73 5935.79 11.48 107.02
		 dm-1 0.00 0.00 57.07 40.84 228.27 163.35 8.00 93.84 979.61 13.94 2329.08 10.93 107.02


	上述示例的iostat命令将每2秒打印出报告，共打印5次；-x参数告诉iostata打印出更详尽的报告
	
	iostat打印出的第1个报告，数值是基于最后一次系统启动的时间统计的；基于这个原因，在大部份情况下，iostat打印出的第1个报告应该被忽略。每个子报告都是基于上1次的报告。在这个例子中，我们的命令将打印5次报告，第2份报告就是从第1份报告开始后的硬盘数据，第3份报告基于第2份，依此类推。
	
	上述示例，sda盘的%utilized达到了111.41%。这表示引起I/O慢的进程在写入sda盘。因为我这个测试实例中只有1个硬盘，但对于有多硬盘的服务器来说，这可以缩小在使用I/O的进程范围。
	
	除了iostat的%utilized能提供丰富的信息外，像rrqm/s、wrqm/s这些每秒读、写的请求数，r/s、w/s每秒读写数也很有用。在我们的例子中，我们的程序看起来读写很繁重的信息也能帮助我们确定这个讨人厌的进程  
	
- 查找引起高I/O的进程  

		[root@coolnull ~]# yum install iotop
		[root@coolnull ~]# iotop
		 Total DISK READ: 8.00 M/s | Total DISK WRITE: 20.36 M/s
		  TID PRIO USER DISK READ DISK WRITE SWAPIN IO> COMMAND
		 15758 be/4 root 7.99 M/s 8.01 M/s 0.00 % 61.97 % bonnie++ -n 0 -u 0 -r 239 -s 478 -f -b -d /tmp


	查看哪个进程使用硬盘最多的最简单的方法就是使用iotop命令。通过查看数据，我们很容易就能确定是bonnie++这个进程引起我们机器高I/O
	
	虽然iotop好用，但默认主流的linux发行版中是没有安装的；并且我个人也不推荐依赖默认系统没有安装的命令。系统管理员总是会碰到这样的情况，他们没办法在短时间内简单地安装这些非默认包
	
	如果iotop没办法用，以下的步聚还是可以帮助你缩小这些讨人厌进程的范围  
	
- 进程状态列表

	ps命令能打印出内存，cpu的情况但没办法打印出硬盘I/O的情况。虽然ps没办法打印出I/O的情况，但它可以显示出进程是否在等待I/O
	
	`The ps state field provides the processes current state; below is a list of states from the man page.`  
	
	ps状态列提供了进程当前的状态，以下从man ps上获取的进程stat列表  

		PROCESS STATE CODES
		 D uninterruptible sleep (usually IO)
		 R running or runnable (on run queue)
		 S interruptible sleep (waiting for an event to complete)
		 T stopped, either by a job control signal or because it is being traced.
		 W paging (not valid since the 2.6.xx kernel)
		 X dead (should never be seen)
		 Z defunct ("zombie") process, terminated but not reaped by its parent.  
		 
  等待I/O的进程通过处于uninterruptible sleep或D状态；通过给出这些信息我们就可以简单的查找出处在wait状态的进程  
	
  示例：  
  
		 [root@coolnull ~]# for x in `seq 1 1 10`; do ps -eo state,pid,cmd | grep "^D"; echo "----"; sleep 5; done
		 D 248 [jbd2/dm-0-8]
		 D 16528 bonnie++ -n 0 -u 0 -r 239 -s 478 -f -b -d /tmp
		 ----
		 D 22 [kswapd0]
		 D 16528 bonnie++ -n 0 -u 0 -r 239 -s 478 -f -b -d /tmp
		 ----
		 D 22 [kswapd0]
		 D 16528 bonnie++ -n 0 -u 0 -r 239 -s 478 -f -b -d /tmp
		 ----
		 D 22 [kswapd0]
		 D 16528 bonnie++ -n 0 -u 0 -r 239 -s 478 -f -b -d /tmp
		 ----
		 D 16528 bonnie++ -n 0 -u 0 -r 239 -s 478 -f -b -d /tmp
		 ----


	上述命令会每5秒循环打印出位于D状态的进程，共打印10次
	
	从上面的输出可以看出bonnie++，pid 16528比其它进程更加占用I/O。从这点，bonnie++看起来更有可能引起I/O Wait。但仅凭进程处于uninterruptible sleep state誊，还不能完全确定就是这引起的I/O wait。
	
	为了帮助肯定我们的怀疑，我们可以使用/proc文件系统。在这个进程目录里，每个进程都有一个io文件，里面的数值跟iotop命令获取的I/O数值一样  
	
		 [root@coolnull ~]# cat /proc/16528/io
		 rchar: 48752567
		 wchar: 549961789
		 syscr: 5967
		 syscw: 67138
		 read_bytes: 49020928
		 write_bytes: 549961728
		 cancelled_write_bytes: 0  
		 
	read\_bytes和write\_bytes就是这个进程读写硬盘的字节数。在这里，bonnie++已经读取了46MB，写入524MB的数据。对很多进程，这可能不是很多，但在我们这个实例这足够引起高i/o wait  
	
- 查找哪个文件在被繁重地写入

	lsof命令会为你展示指定进程打开的所有文件或依赖提供选项的所有进程。从这个列表，人们可以根据文件的大小和/proc io文件里出现的次数做出有用的猜测，哪个文件正在被频繁地写入。
	
	为了减少输出的内容，我们可以使用-p 选项来只打印指定进程id打开的文件  
	
		 [root@coolnull ~]# lsof -p 16528
		 COMMAND PID USER FD TYPE DEVICE SIZE/OFF NODE NAME
		 bonnie++ 16528 root cwd DIR 252,0 4096 130597 /tmp
		 <truncated>
		 bonnie++ 16528 root 8u REG 252,0 501219328 131869 /tmp/Bonnie.16528
		 bonnie++ 16528 root 9u REG 252,0 501219328 131869 /tmp/Bonnie.16528
		 bonnie++ 16528 root 10u REG 252,0 501219328 131869 /tmp/Bonnie.16528
		 bonnie++ 16528 root 11u REG 252,0 501219328 131869 /tmp/Bonnie.16528
		 bonnie++ 16528 root 12u REG 252,0 501219328 131869 <strong>/tmp/Bonnie.16528

  为了更进一步确认是这些文件被繁重地写入，我们可以看下/tmp文件系统是不是sda盘的一部份  
  
		 [root@coolnull ~]# df /tmp
		 Filesystem 1K-blocks Used Available Use% Mounted on
		 /dev/mapper/workstation-root 7667140 2628608 4653920 37% /  
		 
   从df的输出我们可以判断出/tmp是根目录下的一部份  

