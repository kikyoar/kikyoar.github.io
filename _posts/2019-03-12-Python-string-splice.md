---
layout:     post
title:      "python3 拼接字符串的7种方法"
subtitle:   "字符串的高级玩法"
date:       2019-03-12
author:     "kikyoar"
header-img: "img/post-bg-python-version.jp"
tags:
    - Python
---  

[摘自博客园](https://www.cnblogs.com/Jimc/p/9634427.html)

- 直接通过（+）操作符拼接  
	
		'Hello' + ' ' + 'World' + '!'
		'Hello World!'  

	使用这种方式进行字符串连接的操作效率低下，因为python中使用 + 拼接两个字符串时会生成一个新的字符串，生成新的字符串就**需要重新申请内存**，当拼接字符串较多时自然会影响效率  

- 通过str.join()方法拼接

		strlist = ['Hello', ' ', 'World', '!']
		''.join(strlist)
		'Hello World!'

	这种方式一般常使用在**将集合转化为字符串，''.join()其中''可以是空字符，也可以是任意其他字符，当是任意其他字符时，集合中字符串会被该字符隔开**   

- 通过str.format()方法拼接  

		'{} {}!'.format('Hello', 'World')
		'Hello World!'

	通过这种方式拼接字符串需要注意的是字符串中**{}的数量要和format方法参数数量一致**，否则会报错  

- 通过(%)操作符拼接  
		
		'%s %s!' % ('Hello', 'World')
		'Hello World!'

	这种方式与str.format()使用方式基本一致  

- 通过()多行拼接
		
		  (
		...     'Hello'
		...     ' '
		...     'World'
		...     '!'
		... )
		
		'Hello World!'


	**python遇到未闭合的小括号，自动将多行拼接为一行** 

- 通过string模块中的Template对象拼接  

		from string import Template
		s = Template('${s1} ${s2}!') 
		s.safe_substitute(s1='Hello',s2='World')
		'Hello World!'  


	Template的实现方式是首先通过Template初始化一个字符串。这些字符串中包含了一个个key。通过**调用substitute或safe_subsititute**，将key值与方法中传递过来的参数对应上，从而实现在指定的位置导入字符串。这种方式的好处是不需要担心参数不一致引发异常，如：

		from string import Template
		s = Template('${s1} ${s2} ${s3}!') 
		s.safe_substitute(s1='Hello',s2='World')
		'Hello World ${s3}!'

- 通过F-strings拼接

	在python3.6.2版本中，PEP 498 提出一种新型字符串格式化机制，被称为“字符串插值”或者更常见的一种称呼是F-strings，F-strings提供了一种明确且方便的方式将python表达式嵌入到字符串中来进行格式化：

		s1 = 'Hello'
		s2 = 'World'
		f'{s1} {s2}!'
		'Hello World!'

	在F-strings中我们也可以执行函数：

		def power(x):
		...     return x*x
		... 
		x = 5
		f'{x} * {x} = {power(x)}'
		'5 * 5 = 25'

	**而且F-strings的运行速度很快，比%-string和str.format()这两种格式化方法都快得多**  