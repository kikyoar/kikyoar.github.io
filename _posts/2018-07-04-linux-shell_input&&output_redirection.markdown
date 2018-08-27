---
layout:     post
title:      "linux shell数据重定向详细分析"
subtitle:   "输入重定向与输出重定向"
date:       2018-07-04
author:     "kikyoar"
header-img: "img/post-bg-linux-version.jpg"
tags:
    - Linux
---  


**linux文件描述符：**可以理解为linux跟踪打开文件，而分配的一个数字，这个数字有点类似c语言操作文件时候的句柄，通过句柄就可以实现文件的读写操作。 用户可以自定义文件描述符范围是：3-num,这个最大数字，跟用户的：ulimit –n 定义数字有关系，不能超过最大值  

**linux启动后，会默认打开3个文件描述符，分别是：标准输入standard input 0,正确输出standard output 1,错误输出：error output 2**  

**以后打开文件后。新增文件绑定描述符 可以依次增加。 一条shell命令执行，都会继承父进程的文件描述符。因此，所有运行的shell命令，都会有默认3个文件描述符**

**文件输入输出由追踪为一个给定的进程所有打开文件的整数句柄来完成。这些数字值就是文件描述符。最为人们所知的文件描述符是 stdin, stdout 和 stderr，文件描述符的数字分别是0，1和2。这些数字和各自的设备是保留的。一个命令执行前，先会准备好所有输入输出，默认分别绑定（stdin,stdout,stderr)，如果这个时候出现错误，命令将终止，不会执行**  

**这些默认的输出，输入都是linux系统内定的，我们在使用过程中，有时候并不希望执行结果输出到屏幕。我想输出到文件或其它设备。这个时候我们就需要进行输出重定向了**  

Linux shell下常用输入输出操作符是：  

- 标准输入   (stdin) ：代码为 0 ，使用 < 或 << ； /dev/stdin -> /proc/self/fd/0   0代表：/dev/stdin 
- 标准输出   (stdout)：代码为 1 ，使用 > 或 >> ； /dev/stdout -> /proc/self/fd/1  1代表：/dev/stdout
- 标准错误输出(stderr)：代码为 2 ，使用 2> 或 2>> ； /dev/stderr -> /proc/self/fd/2 2代表：/dev/stderr  

## 输出重定向：  

**格式：**

	command-line1 [1-n] > file或文件操作符或设备

将一条命令执行结果（标准输出，或者错误输出，本来都要打印到屏幕上面的）  重定向其它输出设备（文件，打开文件操作符，或打印机等等）1,2分别是标准输出，错误输出  

**实例：**
	
	显示当前目录文件 test.sh test1.sh test1.sh实际不存在
	
	[chengmo@centos5 shell]$ ls test.sh test1.sh
	ls: test1.sh: 没有这个文件和目录
	test.sh
	 
	正确输出与错误输出都显示在屏幕了，现在需要把正确输出写入suc.txt
	1>可以省略，不写，默认所至标准输出
	
	[chengmo@centos5 shell]$ ls test.sh test1.sh 1>suc.txt
	ls: test1.sh: 没有这个文件和目录
	[chengmo@centos5 shell]$ cat suc.txt 
	test.sh
	 
	把错误输出，不输出到屏幕，输出到err.txt
	
	[chengmo@centos5 shell]$ ls test.sh test1.sh 1>suc.txt 2>err.txt
	[chengmo@centos5 shell]$ cat suc.txt err.txt 
	test.sh
	ls: test1.sh: 没有这个文件和目录
	
	继续追加把输出写入suc.txt err.txt  “>>”追加操作符
	
	[chengmo@centos5 shell]$ ls test.sh test1.sh 1>>suc.txt 2>>err.txt 
	 
	将错误输出信息关闭掉
	
	[chengmo@centos5 shell]$ ls test.sh test1.sh 2>&-
	test.sh
	[chengmo@centos5 shell]$ ls test.sh test1.sh 2>/dev/null
	test.sh
	
	&[n] 代表是已经存在的文件描述符，&1 代表输出 &2代表错误输出 &-代表关闭与它绑定的描述符
	/dev/null 这个设备，是linux 中黑洞设备，什么信息只要输出给这个设备，都会给吃掉 
	关闭所有输出
	
	[chengmo@centos5 shell]$ ls test.sh test1.sh  1>&- 2>&- 
	关闭 1 ，2 文件描述符
	
	[chengmo@centos5 shell]$ ls test.sh test1.sh  2>/dev/null 1>/dev/null
	将1,2 输出转发给/dev/null设备 
	
	[chengmo@centos5 shell]$ ls test.sh test1.sh >/dev/null 2>&1  	
	将错误输出2 绑定给 正确输出 1，然后将正确输出发送给 /dev/null设备这种常用
	[chengmo@centos5 shell]$ ls test.sh test1.sh &>/dev/null
	#& 代表标准输出 ，错误输出将所有标准输出与错误输出输入到/dev/null文件  
	
	
**注意：**

- shell遇到”>”操作符，会判断右边文件是否存在，如果存在就先删除，并且创建新文件。不存在直接创建。 无论左边命令执行是否成功。右边文件都会变为空

- “>>”操作符，判断右边文件，如果不存在，先创建。以添加方式打开文件，会分配一个文件描述符[不特别指定，默认为1,2]然后，与左边的标准输出（1）或错误输出（2） 绑定

- 当命令：执行完，绑定文件的描述符也自动失效。0,1,2又会空闲

- 一条命令启动，命令的输入，正确输出，错误输出，默认分别绑定0,1,2文件描述符

- 一条命令在执行前，先会检查输出是否正确，如果输出设备错误，将不会进行命令执行  

## 输入重定向：  

**格式：**

	command-line [n] <file或文件描述符&设备

命令默认从键盘获得的输入，改成从文件，或者其它打开文件以及设备输入。执行这个命令，将标准输入0，与文件或设备绑定。将由它进行输入  

**实例：**  

	[chengmo@centos5 shell]# cat > catfile 
	testing 
	cat file test
	
	这里按下 [ctrl]+d 离开 
	从标准输入【键盘】获得数据，然后输出给catfile文件
	 
	[chengmo@centos5 shell]$ cat>catfile <test.sh
	cat 从test.sh 获得输入数据，然后输出给文件catfile
	 
	 
	[chengmo@centos5 shell]$ cat>catfile <<eof
	test a file
	test!
	eof
	 
	<< 这个连续两个小符号， 他代表的是『结束的输入字符』的意思。这样当空行输入eof字符，输入自动结束，不用ctrl+D  
	
## exec绑定重定向：  

**格式：**
	
	exec 文件描述符[n] <或> file或文件描述符或设备

在上面讲的输入，输出重定向 将输入，输出绑定文件或设备后。只对当前那条指令是有效的。如果需要在绑定之后，接下来的所有命令都支持的话。就需要用exec命令  


**实例：** 
	
	[chengmo@centos5 shell]$ exec 6>&1
	将标准输出与fd 6绑定
	 
	[chengmo@centos5 shell]$ ls  /proc/self/fd/ 
	0  1  2  3  6
	出现文件描述符6
	 
	[chengmo@centos5 shell]$ exec 1>suc.txt
	将接下来所有命令标准输出，绑定到suc.txt文件（输出到该文件）
	 
	[chengmo@centos5 shell]$ ls -al
	执行命令，发现什么都不返回了，因为标准输出已经输出到suc.txt文件了
	 
	[chengmo@centos5 shell]$ exec 1>&6
	恢复标准输出
	 
	 
	[chengmo@centos5 shell]$ exec 6>&-
	关闭fd 6描述符
	 
	[chengmo@centos5 ~]$ ls /proc/self/fd/
	0  1  2  3  

**说明：使用前先将标准输入保存到文件描述符6，这里说明下，文件描述符默认会打开0,1,2 还可以使用自定义描述符 。然后对标准输出绑定到文件，接下来所有输出都会发生到文件。 使用完后，恢复标准的输出，关闭打开文件描述符6**  

**复杂实例：** 

	exec 3<>test.sh;
	打开test.sh可读写操作，与文件描述符3绑定
	 
	while read line<&3
	 do
	    echo $line;
	done
	循环读取文件描述符3（读取的是test.sh内容）
	exec 3>&-
	exec 3<&-
	关闭文件的，输入，输出绑定