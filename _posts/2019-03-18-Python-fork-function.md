---
layout:     post
title:      "fork()创建多进程"
subtitle:   "os.fork多进程详解"
date:       2019-03-18
author:     "kikyoar"
header-img: "img/post-bg-python-version.jp"
tags:
    - Python
---  

[摘自CSDN](http://www.cnblogs.com/gengyi/p/8564052.html)

##  进程概述  
### 进程  

进程是程序的一次动态执行过程，它对应了从代码加载、执行到执行完毕的一个完整过程  

进程是系统进行资源分配和调度的一个独立单位。进程是由代码（堆栈段）、数据（数据段）、内核状态和一组寄存器组成  

在多任务操作系统中，通过运行多个进程来并发地执行多个任务。由于每个线程都是一个能独立执行自身指令的不同控制流，因此一个包含多个线程的进程也能够实现进程内多任务的并发执行  
 
进程是一个内核级的实体，进程结构的所有成分都在内核空间中，一个用户程序不能直接访问这些数据  

**进程的状态： 创建、准备、运行、阻塞、结束**  

### 进程间的通信方式
- 文件  
- 管道  
- socket  
- 信号  
- 信号量  
- 共享内存

### 进程的创建函数  

- os.fork()

- subprocess

- processing

- multiprocessing  


## 创建进程  

### 子进程的创建方式
Linux 和 Unix 操作系统提供了一个fork()函数创建新的进程，这也就意为**这该函数仅适用于Linux和Unix平台**   

fork()函数比较特殊，**python的os.fork()是唯一调用一次返回两次的函数，因操作系统将当前的进程（父进程）复制了一份新的进程（子进程），然后分别在父进程和子进程内返回**  

例如：采用父子进程拷贝文件，子进程的拷贝并不是从开始就就行的（除非进行设置），混乱拷贝文件可能会出现结果异常而出现错误  

**注意：有时在一个终端会混合出现打印结果**  

fork()从本质上属于内建函数，通过 os 模块导入，从函数原型上看，**子进程永远返回0，而父进程则返回子进程的PID**。这意味着每个子进程的创建都会在父进程中留下标记PID（子进程身份信息），当子进程溯源其父进程时，通过getppid()就可以找到父进程的PID  

**fork()语法**：

	功能：为当前进程创建一个子进程
	
	参数：无
	
	返回值：0 和 子进程PID（在父进程中）
	
	　　　　< 0 子进程创建失败
	
	　　　　= 0 在子进程中的返回值
	
	　　　　> 0 在父进程中的返回值

特点：

	（1）子进程会继承父进程几乎全部代码段（包括fork()前所定义的所有内容）
	
	（2）子进程拥有自己独立的信息标识，如PID
	
	（3）父、子进程独立存在，在各自存储空间上运行，互不影响
	
	（4）创建父子进程执行不同的内容是多任务中固定方法
	
### 创建简单的子进程： 

	import os
	pid = os.fork()
	
	if pid <0 :
	    print("create process failed")
	elif pid == 0:
	    print("this is child process")
	else :
	    print("this is parent process")
	
	print("******the end*******")  
	
输出：

	this is parent process
	******the end*******
	this is child process
	******the end*******
	#或
	this is parent process
	this is child process
	******the end*******
	******the end*******  
	
注：

（a）**前后不一致的原因，因为子进程和父进程独立运行，但是打印在同一个终端上，所以终端上父子进程同时打印出来**  

（b）**子进程运行时是从 pid = os.fork() 下面语句执行，实际上，该语句是两条语句， os.frok() 是创建子进程语句，而 pid =  是赋值语句，所以在创建完子进程后，下一句为运行赋值语句**  

（c）**该语句中，父进程先运行，运行中，子进程也运行，但此时并不代表父进程已经运行结束；所以出现终端语句混乱**  

### 添加时间调整执行顺序  

	import os
	from time import sleep
	
	pid = os.fork()
	if pid <0 :
	    print("create process failed")
	elif pid == 0:
	    sleep(3)
	    print("this is child process")
	else :
	    sleep(1)
	    print("this is parent process")
	
	print("******the end*******")

输出：
	
	this is parent process
	******the end*******
	this is child process
	******the end*******  
	
###  变量在父子进程中的关系  

	import os
	from time import sleep
	
	a = 10 #父子进程中含有相同的资源，包括fork()之前的语句
	pid  = os.fork()
	
	if pid < 0:
	    print('create process failed')
	elif pid == 0:
	    a = 1000 #子进程将a值修改为1000
	    print("This is child process")
	else:
	    sleep(2)
	    print('a = ',a)
	    print('The pid = ',pid)
	    print("This is parent process")
	
	print('a = ',a)
	print("**********the end***********")  

输出：	

	
	This is child process
	a =  1000
	**********the end***********
	a =  10
	The pid =  5495
	This is parent process
	a =  10
	**********the end***********
注：

**在父进程中变量的变量a=10，而在子进程中将其绑定为1000时，其在父进程中的绑定值并没有变化。这主要是因父子进程中位于不同的虚拟空间所致**  


### 父子进程PID  

**os.getpid() 获取当前进程的PID号**

**os.getppid() 获取当前进程父进程的PID号**  

	import os
	from time import sleep
	
	pid = os.fork()
	
	if pid < 0:
	    print("create process failed")
	elif pid == 0:
	    print("my PID is",os.getpid()) #子进程PID
	    print("my parent process-1 PID is",os.getppid()) #父进程PID
	    sleep(5)
	    print("my parent process-2 PID is",os.getppid()) #父进程PID
	    print("this is child process")
	else:
	    sleep(1)
	    print("===========================")
	    print("the PID is",pid) #父进程中fork()返回值　＝　子进程PID
	    print("the parent PID is",os.getpid()) #父进程PID
	    print("this is parent process")


输出：

	my PID is 4338
	my parent process-1 PID is 4337
	===========================
	the PID is 4338
	the parent PID is 4337
	this is parent process
	my parent process-2 PID is 2236
	this is child process 
	
注：在子进程中，sleep(5)前后的子进程对应的父进程是不一样的（PID不一致），这是由于当父进程结束后，子进程就成了孤儿进程  

孤儿进程将被init进程(该进程号即为子进程新的父进程PID)所收养，并由init进程对它们完成状态收集工作  


**fork  侧重于父子进程都有任务，适合于少量的子进程创建**	
