---
layout:     post
title:      "linux下 文件夹和文件的字符集编码方式转换"
subtitle:   "Linux中文显示乱码"
date:       2018-06-01
author:     "kikyoar"
header-img: "img/post-bg-linux-version.jpg"
tags:
    - Linux
---   


 
**Linux下部分中文乱码**   
用[convmv](https://www.j3e.de/linux/convmv/convmv-1.15.tar.gz)工具将Resources目录下的所有文件的名称使用utf-8重新编码

**若用ftp客户端访问资源时，遇到乱码情况，也请核实客户端编码方式和服务器是否一致**  

	./convmv -f GB2312  -t UTF-8 -r --notest /Resources/*  

以上讲述了在linux下修改目录名编码的方法，再扩展一下文件内容更改编码的方法  

- 在Vim中直接进行转换文件编码,比如将一个文件转换成utf-8格式:  

		:set fileencoding=utf-8

- iconv 转换，iconv的命令格式如下：  

		iconv -f encoding -t encoding inputfile

	比如将一个UTF-8 编码的文件转换成GBK编码  

		iconv -f GBK -t UTF-8 file1 -o file2  
		
**Linux下全部中文显示乱码**  

如果我在系统中安装了不同的语言包和不同的字体，系统是如何判断我所要的语言界面并调用相关的字体的呢？ 系统中那些文件和变量在控制这些呢？   

可以使用locale命令，查看当前系统默认采用的字符集  

	
	# locale   
	
在redHat/CentOS系统下，记录系统默认使用语言的文件是/etc/sysconfig/i18n，如果默认安装的是中文的系统，i18n的内容如下：   

	LANG="zh_CN.UTF-8" 
	SYSFONT="latarcyrheb-sun16" 
	SUPPORTED="zh_CN.UTF-8:zh_CN:zh"  
	
- 其中LANG变量是language的简称，这个变量是决定系统的默认语言的，即系统的菜单、程序的工具栏语言、输入法默认语言等。

- SYSFONT是system font的简称，决定系统默认用哪一种字体。

- SUPPORTED变量决定系统支持的语言，即系统能够显示的语言。

**安装中文语言包与编码设置**  

- 系统必须安装中文语言包才行（中文字体）  

		# yum -y groupinstall chinese-support   
		
- 仅仅有语言包还不行，我们得设置相应的字符集编码  

		## 临时生效
		# export LANG="zh_CN.UTF-8"    # 设置为中文
		# export LANG="en_US.UTF-8"    # 设置为英文，我比较喜欢这样 export LANG=C
		 
		## 永久生效， 编辑/etc/sysconfig/i18n（最好reboot一下）
		LANG="zh_CN.UTF-8"
		 
		 
		## 或者编辑 /etc/profile 配置文件，添加如下一行
		export LANG="zh_CN.UTF-8"
		# 重新载入
		. /etc/profile
		 
		## 查看当前的字符集
		echo $LANG   
		


  