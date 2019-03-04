---
layout:     post
title:      "python中copy和deepcopy的使用方法"
subtitle:   "浅拷贝与深拷贝"
date:       2019-03-04
author:     "kikyoar"
header-img: "img/post-bg-python-version.jp"
tags:
    - Python
--- 


* python2中，需要import copy模块  
* python3中，直接可以使用copy()方法，但deepcopy()还是需要导入copy模块  

下面以python2为例

	import copy
	
	list = ['beijing', 'tianjin', 'hebei', 'wuhan', 'shandong']
	list_copy = copy.copy(list)
	
	print(id(list))
	print(id(list_copy))
	
	list[0] = 'heilongjiang'
	
	print(list)
	print(list_copy)


运行结果：

	4440174152
	4435733320
	['heilongjiang', 'tianjin', 'hebei', 'wuhan', 'shandong']
	['beijing', 'tianjin', 'hebei', 'wuhan', 'shandong']


**list改变了，list_copy没有跟着改变,那如果list里面包含了子列表呢**

	import copy
	list = ['beijing','tianjin','hebei',['neimeng','xinjiang'],'wuhan','shandong']
	list_copy = copy.copy(list)
	list[3][0] = 'taiwan'
	print(list)
	print(list_copy)



结果显示：

	['beijing', 'tianjin', 'hebei', ['taiwan', 'xinjiang'], 'wuhan', 'shandong']
	['beijing', 'tianjin', 'hebei', ['taiwan', 'xinjiang'], 'wuhan', 'shandong']



**为什么结果跟着变了呢，因为copy为浅copy，只复制了第一层数据，列表里存储的子列表，打印出来是子列表，其实，在内存里，列表里只是存储了子列表的内存地址，子列表在内存里是单独存储的改变了子列表，再打印list_copy时，子列表内存地址地址没有变，打印出来自然是修改后的子列表**


浅copy的实现方法：

	l1 = list[:]
	l2 = copy.copy(list)
	l3 = list(list)

那浅copy的用处呢,比如两口子，共有一个账号存款

	card = ['name',['saving',100]]
	husband = copy.copy（card）
	wife = copy.copy(card)
	husband[0]= 'zhangsan'
	wife[0]='fengjie'
	husband[1][1] = 50 #丈夫取出50，还剩下50
	print husband
	print wife

结果显示：

	['zhangsan', ['saving', 50]]
	['fengjie', ['saving', 50]]

**两个人的账号存款同时变动,那能不能完全copy呢，使用命令deepcopy就可以**

	import copy
	
	list = ['beijing','tianjin','hebei',['neimeng','xinjiang'],'wuhan','shandong']
	list_copy = copy.deepcopy(list)
	list[3][0] = 'taiwan'
	print(list)
	print(list_copy)

结果显示：

	['beijing', 'tianjin', 'hebei', ['taiwan', 'xinjiang'], 'wuhan', 'shandong']
	['beijing', 'tianjin', 'hebei', ['neimeng', 'xinjiang'], 'wuhan', 'shandong']

这样复制就不会改变子列表的值了，是因为deepcopy将子列表也复制了一份

**注：不过，deepcopy方法，如果数据很大，完全复制就是存储两份数据，占用内存，慎用！**

**copy 对于不可变类型(元组等)为浅拷贝,对于可变类型(列表等)为深拷贝**

