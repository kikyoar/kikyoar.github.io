---
layout:     post
title:      "Shell脚本字符串单引号和双引号的区别浅析"
subtitle:   "常用的字符串相关方法"
date:       2019-03-11
author:     "kikyoar"
header-img: "img/post-bg-linux-version.jpg"
tags:
    - Linux
--- 

[摘自脚本之家](https://www.jb51.net/article/52379.htm)


字符串是shell编程中最常用最有用的数据类型（除了数字和字符串，也没啥其它类型好用了），字符串可以用单引号，也可以用双引号，也可以不用引号。单双引号的区别跟PHP类似  

**单引号**  

	str='this is a string'

单引号字符串的限制：

- **单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的**
- **单引号字串中不能出现单引号（对单引号使用转义符后也不行）**  


**双引号**


	your_name='qinjx'
	str="Hello, I know your are \"$your_name\"! \n"

双引号的优点：

- **双引号里可以有变量**
- **双引号里可以出现转义字符**  

**反引号**   

命令替换是指shell能够将一个命令的标准输出插在一个命令行中任何位置。shell中有两种方法作命令替换：把shell命令用反引号或者\$(...)结构括起来，其中，\$(...)格式受到POSIX标准支持，也利于嵌套  

	echo The date and time is `date` 
	#The date and time is 三 6月 15 06:10:35 CST 2005 
	echo Your current working directory is $(pwd) 
	#Your current working directory is /home/howard/script 


**反斜杠**

反斜杠一般用作转义字符，或称逃脱字符，Linux如果echo要让转义字符发生作用，就要使用-e选项，且转义字符要使用双引号 
	
	echo -e "\n" 

反斜杠的另一种作用，就是当反斜杠用于一行的最后一个字符时，Shell把行尾的反斜杠作为续行，这种结构在分几行输入长命令时经常使用



## 常用的字符串相关方法

**拼接字符串**  

	your_name="qinjx"
	greeting="hello, "$your_name" !"
	greeting_1="hello, ${your_name} !"
	echo $greeting $greeting_1

**获取字符串长度**  


	string="abcd"
	echo ${#string} #输出 4


**提取子字符串**  

	string="alibaba is a great company"
	echo ${string:1:4} #输出liba

**查找子字符串**  


	string="alibaba is a great company"
	echo `expr index "$string" is`  
	

