---
layout:     post
title:      "关于Centos7中无法使用rpm和yum的故障处理总结"
subtitle:   "动态库造成的故障详解"
date:       2019-04-21
author:     "kikyoar"
header-img: "img/post-bg-linux-version.jpg"
tags:
    - Linux
---  


### 故障描述：
由于现场执行以下俩条命令后rpm和yum命令无法使用
	
	rpm -e python-devel-2.7.5-58.el7.x86_64
	rpm -e gnome-python2-canvas-2.28.1-14.el7.x86_64 --nodeps

报错为： 

![load1](http://kikyoar.com/img/load1.png)  

### 处理过程：

- 思路一：  
根据报错Centos7 error: Failed to initialize NSS library，查询google上大部分给出的解决方法为：
 
	- 1、下载[nspr(nspr-4.13.1-1.0.el7_3.x86_64.rpm)](http://mirror.centos.org/centos/7/os/x86_64/Packages/nspr-4.13.1-1.0.el7_3.x86_64.rpm)  
	- 2、执行命令：       	
			`rpm2cpio nspr-4.13.1-1.0.el7_3.x86_64.rpm | cpio –idmv`  
   - 3、执行命令：    
          `LD_PRELOAD=./usr/lib64/libnspr4.so yum update nspr`  
 
 	
 	**以上执行操作完成后，发现未解决问题**  

- 思路二：  

	由于是删除qt，python-devel-2.7.5-58.el7.x86_64，gnome-python2-canvas-2.28.1，所以感觉是文件丢失，于是查看同一批主机中的包的版本：    
`rpm -qa | grep 相关软件名(gnome-python2、python-devel、python-rpm、python等)`
然后从官方网站下载依赖包和软件执行，**此操作也无效，未解决问题**  

- 思路三：  
	经比对rpm和yum包的依赖关系，发现yum依赖于rpm，也就是说rpm如果好了的话，yum也会好，因此如果从其他主机拷贝文件的方法无法解决，则应该考虑编译安装rpm，查看其他主机rpm版本为4.11.3，因此从rpm 官网下载源码包进行安装，但是由于是编译环境，因此需要解决依赖问题，以下为注意事项：
	- 1、解决依赖关系：  
	rpm 依赖的包：**nss  nss-devel  nspr  nspr-devel  file  file-devel  popt  popt-devel  db4  db4-devel  lua-static  lua-devel** ,如果以上未安装的话则会出现以下报错：
		- 如果出现configure: error: missing required NSPR / NSS header，首先要编译安装**nss nss-devel nspr nspr-devel**
		- 如果出现configure: error: missing required header magic.h，出现此错误是因为找不到magic的头文件,安装**file和其开发包即可** 
		- 如果出现configure: error: missing required header popt.h,出现此错误是因为找不到popt的头文件,安装**popt和其开发包即可** 
		- 如果出现configure: error: internal Berkeley DB directory not present,出现此错误是因为找Berkeley db目录不存在,安装**db4和其开发包即可**   

	- 2、以上软件包在编译中，可能会一直报其他依赖问题，比如nss缺失依赖等等现象，现场应该根据报错解决，但是此解决办法较为费时间  

- 思路四：  
根据故障机的原有安装环境，搭建虚拟机，模拟操作，现在主要是不明确在输入`rpm -e gnome-python2-canvas-2.28.1-14.el7.x86_64`中所报的依赖包报错是哪一个，而直接用—nodeps卸载了，如果明确了报错，也可以方便的找出故障点  

### 最终解决办法：
据现场同学提供的思路，以下为解决办法：  

经google后多方查找发现：  
[链接1](https://blog.csdn.net/u011138447/article/details/78131429)    
[链接2](https://bugzilla.redhat.com/show_bug.cgi?id=1515890)    

以上俩个链接均为误删除sqlite3后造成的**“Failed to initialize NSS library”**，通过查看sqlite3的依赖发现  
![load2](http://kikyoar.com/img/load2.png)  

![load3](http://kikyoar.com/img/load3.png)  

**libsqlite3.so** 目前nss-softokn和python都依赖它（sqlite现在在python std库中），所以是强行删除它会破坏很多东西  

再在故障机目录/usr/lib64/上查看是否存在libsqlite3.so，发现并未存在：  

- 1、下载sqlite-3.7.17-8.el7.x86_64
- 2、rpm2cpio sqlite-3.7.17-8.el7.x86_64.rpm | cpio –idmv
- 3、cp  -R  usr/   /
- 4、第三步也可以用  

	`LD_PRELOAD=./usr/lib64/libnspr4.so:./usr/lib64/libnss3.so:./usr/lib64/libsqlite3.so`  

**以上操作故障解决**  

### 下面为测试机做实验结果：  
 
![load4](http://kikyoar.com/img/load4.png)

![load5](http://kikyoar.com/img/load5.png)  


### 附注LD_PRELOAD用法：  

![load6](http://kikyoar.com/img/load6.png) 



