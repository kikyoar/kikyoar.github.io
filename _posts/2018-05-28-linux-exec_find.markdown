---
layout:     post
title:      "Linux exec命令详解"
subtitle:   "exec命令在find中的使用"
date:       2018-05-28
author:     "kikyoar"
header-img: "img/post-bg-linux-version.jpg"
tags:
    - Linux
---   

**exec和source都属于bash内部命令(builtins commands)，在bash下输入man exec或man source可以查看所有的内部命令信息。**  

**bash shell的命令分为两类：外部命令和内部命令。外部命令是通过系统调用或独立的程序实现的，如sed、awk等等。内部命令是由特殊的文件格式(.def)所实现，如cd、history、exec等等。内部命令实际上是shell程序的一部分，其中包含的是一些比较简练的UNIX系统命令，这些命令由shell程 序识别并在shell程序内部完成运行，通常在UNIX系统加载运行时shell就被加载并驻留在系统内存中。外部命令是UNIX系统中的实用程序部分， 因为实用程序的功能通常都比较强大，所以它们包含的程序量也会很大，在系统加载时并不随系统一起被加载到内存中，而是在需要时才将其调进内存。通常外部命 令的实体并不包含在shell中，但是其命令执行过程是由shell 程序控制的。shell程序管理外部命令执行的路径查找、加载存放，并控制命令的执行。**  

**理解了内部命令和外部命令之后，你可能就会有疑问了？？那么我怎么看一个命令是内部命令还是外部命令呢？？**   

1、在终端下输入man builtins，即可调出内部命令的大纲和解释  

2、用命令enable调出来，后者将分行打印出内部命令  

	[root@localhost shell]# enable ls
	
	bash: enable: ls: not a shell builtin  
	
在说明exe和source的区别之前，先说明一下fork的概念

　　**fork是linux的系统调用，用来创建子进程(child process)。子进程是父进程(parent process)的一个副本，从父进程那里获得一定的资源分配以及继承父进程的环境。子进程与父进程唯一不同的地方在于pid(process id)。环境变量(传给子进程的变量，遗传性是本地变量和环境变量的根本区别)只能单向从父进程传给子进程。不管子进程的环境变量如何变化，都不会影响父进程的环境变量**  
　　**shell script:有两种方法执行shell scripts，一种是新产生一个shell，然后执行相应的shell scripts;一种是在当前shell下执行，不再启用其他shell。新产生一个shell然后再执行scripts的方法是在scripts文件开头加入以下语句:**  

	#!/bin/sh  
一般的script文件(.sh)即是这种用法。这种方法先启用新的sub-shell(新的子进程),然后在其下执行命令。另外一种方法就是上面说过的source命令，不再产生新的shell，而在当前shell下执行一切命令  

	source  
source命令即点(.)命令。在bash下输入man source，找到source命令解释处，可以看到解释”Read and execute commands from filename in thecurrent shell environment and …”。从中可以知道，source命令是在当前进程中执行参数文件中的各个命令，而不是另起子进程(或sub-shell)   

	exec  
在bash下输入man exec，找到exec命令解释处，可以看到有”No new process is created.”这样的解释，这就是说exec命令不产生新的子进程。那么exec与source的区别是什么呢?exec命令在执行时会把当前的shell process关闭，然后换到后面的命令继续执行  

- **系统调用exec是以新的进程去代替原来的进程，但进程的PID保持不变。因此，可以这样认为，exec系统调用并没有创建新的进程，只是替换了原来进程上下文的内容。原进程的代码段，数据段，堆栈段被新的进程所代替。一个进程主要包括以下几个方面的内容:**   

	(1) 一个可以执行的程序  
	(2) 与进程相关联的全部数据(包括变量，内存，缓冲区)  
	(3) 程序上下文(程序计数器PC,保存程序执行的位置)   

- **exec是一个函数簇，由6个函数组成，分别是以excl和execv打头的** 

	执行exec系统调用，一般都是这样，用fork()函数新建立一个进程，然后让进程去执行exec调用。我们知道，在fork()建立新进程之后，父进各与子进程共享代码段，但数据空间是分开的，但父进程会把自己数据空间的内容copy到子进程中去，还有上下文也会copy到子进程中去。而为了提高效率，采用一种写时copy的策略，即创建子进程的时候，并不copy父进程的地址空间，父子进程拥有共同的地址空间，只有当子进程需要写入数据时(如向缓冲区写入数据),这时候会复制地址空间，复制缓冲区到子进程中去。从而父子进程拥有独立的地址空间。而对于fork()之后执行exec后，这种策略能够很好的提高效率，如果一开始就copy,那么exec之后，子进程的数据会被放弃，被新的进程所代替  

- **exec与system的区别**  

　　(1) exec是直接用新的进程去代替原来的程序运行，运行完毕之后不回到原先的程序中去  
　　(2) system是调用shell执行你的命令，system=fork+exec+waitpid,执行完毕之后，回到原先的程序中去。继续执行下面的部分  
　　

即：  

**sh：父进程会fork一个子进程，shell script在子进程中执行**  
**source:在原进程中执行，不会fork子进程**  
**exec:在原进程中执行，但是同时会终止原进程**  

**注：使用export会把父进程中的变量向子进程中继承，但是反过来却不行，在子进程中，不管环境如果改变，均不会影响父进程**    	

*下面用一个例子来讲解：*   
1.sh  

	#!/bin/bash  
	A=B  
	echo "PID for 1.sh before exec/source/fork:$"  
	export A  
	echo "1.sh: \$A is $A"  
	case $1 in  
	        exec)  
	                echo "using exec..."  
	                exec ./2.sh ;;  
	        source)  
	                echo "using source..."  
	                . ./2.sh ;;  
	        *)  
	                echo "using fork by default..."  
	                ./2.sh ;;  
	esac  
	echo "PID for 1.sh after exec/source/fork:$"  
	echo "1.sh: \$A is $A"    
	
2.sh  

	#!/bin/bash  
	echo "PID for 2.sh: $"  
	echo "2.sh get \$A=$A from 1.sh"  
	A=C  
	export A  
	echo "2.sh: \$A is $A"   

下面在命令行中去执行  
./1.sh fork  

![Linux_exec_fork](http://kikyoar.com/img/exec_fork.gif)  

可以看到，1.sh是在父进程中执行，2.sh是在子进程中执行的，父进程的PID是5344，而子进程的是5345，当子进程执行完毕后，控制权返回到父进程。同时，在子进程改变环境变量A的值不会影响到父进程  

./1.sh source  

![Linux_exec_source](http://kikyoar.com/img/exec_source.gif)  

由结果可知，1.sh和2.sh都是在同一进程中执行的，PID为5367  

./1.sh exec  

![Linux_exec_exec](http://kikyoar.com/img/exec_exec.gif)   

可知，两个脚本都是在同一进程中执行，但是请注意，使用exec终止了原来的父进程，因此，可以看到这两个命令没有执行  

	echo "PID for 1.sh after exec/source/fork:$"  
	echo "1.sh: \$A is $A"   

****	

**linux系统中find命令之exec使用介绍**   

{}   花括号代表前面find查找出来的文件名     
使用find时，只要把想要的操作写在一个文件里，就可以用exec来配合find查找，很方便的。在有些操作系统中只允许-exec选项执行诸如l s或ls -l这样的命令。大多数用户使用这一选项是为了查找旧文件并删除它们。建议在真正执行rm命令删除文件之前，最好先用ls命令看一下，确认它们是所要删除的文件。 exec选项后面跟随着所要执行的命令或脚本，然后是一对儿{ }，一个空格和一个\，最后是一个分号。为了使用exec选项，必须要同时使用print选项。如果验证一下find命令，会发现该命令只输出从当前路径起的相对路径及文件名 

**（1）; (分号）表示command命令参数的结束，特别强调，对于不同的系统，直接使用分号可能会有不同的意义， 所以使用转义符/在分号前明确说明**  
**（2）{}表示文件名，也就是find前面处理过程中过滤出来的文件，用于command命令进行处理**  



**实例1：ls -l命令放在find命令的-exec选项中**   
**命令：** 

	find . -type f -exec ls -l {} \;  

**输出：**  

	[root@localhost test]# find . -type f -exec ls -l {} \; 
	
	-rw-r--r-- 1 root root 127 10-28 16:51 ./log2014.log
	
	-rw-r--r-- 1 root root 0 10-28 14:47 ./test4/log3-2.log
	
	-rw-r--r-- 1 root root 0 10-28 14:47 ./test4/log3-3.log
	
	-rw-r--r-- 1 root root 0 10-28 14:47 ./test4/log3-1.log
	
	-rw-r--r-- 1 root root 33 10-28 16:54 ./log2013.log
	
	-rw-r--r-- 1 root root 302108 11-03 06:19 ./log2012.log
	
	-rw-r--r-- 1 root root 25 10-28 17:02 ./log.log
	
	-rw-r--r-- 1 root root 37 10-28 17:07 ./log.txt
	
	-rw-r--r-- 1 root root 0 10-28 14:47 ./test3/log3-2.log
	
	-rw-r--r-- 1 root root 0 10-28 14:47 ./test3/log3-3.log
	
	-rw-r--r-- 1 root root 0 10-28 14:47 ./test3/log3-1.log
**上面的例子中，find命令匹配到了当前目录下的所有普通文件，并在-exec选项中使用ls -l命令将它们列出**   

**实例2：在目录中查找更改时间在n日以前的文件并删除它们**  

**命令：**

	find . -type f -mtime +14 -exec rm {} \;  
	
**mtime参数的理解应该如下：**  

**-mtime n 按照文件的更改时间来找文件，n为整数。
n表示文件更改时间距离为n天， -n表示文件更改时间距离在n天以内，+n表示文件更改时间距离在n天以前**  

例如：  
-mtime 0 表示文件修改时间距离当前为0天的文件，即距离当前时间不到1天（24小时）以内的文件。
-mtime 1 表示文件修改时间距离当前为1天的文件，即距离当前时间1天（24小时－48小时）的文件。
-mtime＋1 表示文件修改时间为大于1天的文件，即距离当前时间2天（48小时）之外的文件
-mtime -1 表示文件修改时间为小于1天的文件，即距离当前时间1天（24小时）之内的文件

**为什么-mtime＋1 表示文件修改时间为大于1天的文件，即距离当前时间48小时之外的文件，而不是24小时之外的呢？因为n值只能是整数，即比1大的最近的整数是2,所有-mtime＋1不是比当前时间大于1天（24小时），而是比当前时间大于2天（48小时）**  

**输出：**

	[root@localhost test]# ll
	
	总计 328
	
	-rw-r--r-- 1 root root 302108 11-03 06:19 log2012.log
	
	-rw-r--r-- 1 root root     33 10-28 16:54 log2013.log
	
	-rw-r--r-- 1 root root    127 10-28 16:51 log2014.log
	
	lrwxrwxrwx 1 root root      7 10-28 15:18 log_link.log -> log.log
	
	-rw-r--r-- 1 root root     25 10-28 17:02 log.log
	
	-rw-r--r-- 1 root root     37 10-28 17:07 log.txt
	
	drwxr-xr-x 6 root root   4096 10-27 01:58 scf
	
	drwxrwxrwx 2 root root   4096 10-28 14:47 test3
	
	drwxrwxrwx 2 root root   4096 10-28 14:47 test4
	
	
	
	[root@localhost test]# find . -type f -mtime +14 -exec rm {} \;
	
	[root@localhost test]# ll
	
	总计 312
	
	-rw-r--r-- 1 root root 302108 11-03 06:19 log2012.log
	
	lrwxrwxrwx 1 root root      7 10-28 15:18 log_link.log -> log.log
	
	drwxr-xr-x 6 root root   4096 10-27 01:58 scf
	
	drwxrwxrwx 2 root root   4096 11-12 19:32 test3
	
	drwxrwxrwx 2 root root   4096 11-12 19:32 test4
	
	[root@localhost test]# 


**在shell中用任何方式删除文件之前，应当先查看相应的文件，一定要小心！当使用诸如mv或rm命令时，可以使用-exec选项的安全模式。它将在对每个匹配到的文件进行操作之前提示你** 

**实例3：在目录中查找更改时间在n日以前的文件并删除它们，在删除之前先给出提示**  

**命令：**  

	find . -name "*.log" -mtime +5 -ok rm {} \;

**输出：**

	[root@localhost test]# ll
	
	总计 312
	
	-rw-r--r-- 1 root root 302108 11-03 06:19 log2012.log
	
	lrwxrwxrwx 1 root root      7 10-28 15:18 log_link.log -> log.log
	
	drwxr-xr-x 6 root root   4096 10-27 01:58 scf
	
	drwxrwxrwx 2 root root   4096 11-12 19:32 test3
	
	drwxrwxrwx 2 root root   4096 11-12 19:32 test4
	
	[root@localhost test]# find . -name "*.log" -mtime +5 -ok rm {} \;
	
	< rm ... ./log_link.log > ? y
	
	< rm ... ./log2012.log > ? n
	
	[root@localhost test]# ll
	
	总计 312
	
	-rw-r--r-- 1 root root 302108 11-03 06:19 log2012.log
	
	drwxr-xr-x 6 root root   4096 10-27 01:58 scf
	
	drwxrwxrwx 2 root root   4096 11-12 19:32 test3
	
	drwxrwxrwx 2 root root   4096 11-12 19:32 test4
	
	[root@localhost test]#


**在上面的例子中， find命令在当前目录中查找所有文件名以.log结尾、更改时间在5日以上的文件，并删除它们，只不过在删除之前先给出提示。 按y键删除文件，按n键不删除**   

 

**实例4：-exec中使用grep命令**  

**命令：**

	find /etc -name "passwd*" -exec grep "root" {} \;

**输出：**

	[root@localhost test]# find /etc -name "passwd*" -exec grep "root" {} \;
	
	root:x:0:0:root:/root:/bin/bash
	
	root:x:0:0:root:/root:/bin/bash
	
	[root@localhost test]#


**任何形式的命令都可以在-exec选项中使用。  在上面的例子中我们使用grep命令。find命令首先匹配所有文件名为“ passwd\*”的文件，例如passwd、passwd.old、passwd.bak，然后执行grep命令看看在这些文件中是否存在一个root用户**  


**实例5：查找文件移动到指定目录** 

**命令：**

	find . -name "*.log" -exec mv {} .. \;

**输出：**

	[root@localhost test]# ll
	
	总计 12drwxr-xr-x 6 root root 4096 10-27 01:58 scf
	
	drwxrwxr-x 2 root root 4096 11-12 22:49 test3
	
	drwxrwxr-x 2 root root 4096 11-12 19:32 test4
	
	[root@localhost test]# cd test3/
	
	[root@localhost test3]# ll
	
	总计 304
	
	-rw-r--r-- 1 root root 302108 11-03 06:19 log2012.log
	
	-rw-r--r-- 1 root root     61 11-12 22:44 log2013.log
	
	-rw-r--r-- 1 root root      0 11-12 22:25 log2014.log
	
	[root@localhost test3]# find . -name "*.log" -exec mv {} .. \;
	
	[root@localhost test3]# ll
	
	总计 0[root@localhost test3]# cd ..
	
	[root@localhost test]# ll
	
	总计 316
	
	-rw-r--r-- 1 root root 302108 11-03 06:19 log2012.log
	
	-rw-r--r-- 1 root root     61 11-12 22:44 log2013.log
	
	-rw-r--r-- 1 root root      0 11-12 22:25 log2014.log
	
	drwxr-xr-x 6 root root   4096 10-27 01:58 scf
	
	drwxrwxr-x 2 root root   4096 11-12 22:50 test3
	
	drwxrwxr-x 2 root root   4096 11-12 19:32 test4
	
	[root@localhost test]#

**实例6：用exec选项执行cp命令** 

**命令：**

	find . -name "*.log" -exec cp {} test3 \;

**输出：**

	[root@localhost test3]# ll
	
	总计 0
	[root@localhost test3]# cd ..
	
	[root@localhost test]# ll
	
	总计 316
	
	-rw-r--r-- 1 root root 302108 11-03 06:19 log2012.log
	
	-rw-r--r-- 1 root root     61 11-12 22:44 log2013.log
	
	-rw-r--r-- 1 root root      0 11-12 22:25 log2014.log
	
	drwxr-xr-x 6 root root   4096 10-27 01:58 scf
	
	drwxrwxr-x 2 root root   4096 11-12 22:50 test3
	
	drwxrwxr-x 2 root root   4096 11-12 19:32 test4
	
	[root@localhost test]# find . -name "*.log" -exec cp {} test3 \;
	
	cp: “./test3/log2014.log” 及 “test3/log2014.log” 为同一文件
	
	cp: “./test3/log2013.log” 及 “test3/log2013.log” 为同一文件
	
	cp: “./test3/log2012.log” 及 “test3/log2012.log” 为同一文件
	
	[root@localhost test]# cd test3
	
	[root@localhost test3]# ll
	
	总计 304
	
	-rw-r--r-- 1 root root 302108 11-12 22:54 log2012.log
	
	-rw-r--r-- 1 root root     61 11-12 22:54 log2013.log
	
	-rw-r--r-- 1 root root      0 11-12 22:54 log2014.log
	
	[root@localhost test3]#

 
 **find还能一次执行多个 -exec 参数命令**  
 
	mkdir -p a/b/c/ # 先创建测试用例
	find . -name b -exec mkdir -p {}/1/2/3/ \; -exec touch {}/1/hello \;
	ls a/b/1/ # 可以看到有新创建的hello文件
	# 但是这样的话，则会报错。-exec执行的先后就是，命令出现的先后。
	find -name -b -exec touch {}/0/world \; -exec mkdir {}/0 \;  
  

 


