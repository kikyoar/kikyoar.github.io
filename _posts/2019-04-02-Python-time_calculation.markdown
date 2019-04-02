---
layout:     post
title:      "Python程序执行时间的计算方法"
subtitle:   "datetime,time的时间计算方法"
date:       2019-04-02
author:     "kikyoar"
header-img: "img/post-bg-python-version.jp"
tags:
    - Python
---  

[摘自CSDN](https://blog.csdn.net/wangshuang1631/article/details/54286551)

首先说一下我遇到的坑，生产上遇到的问题，我调度Python脚本执行并监控这个进程，python脚本运行时间远远大于python脚本中自己统计的程序执行时间  
监控python脚本执行的时间是36个小时，而python脚本中统计自己执行的时间是4个小时左右.问题暴漏之后首先想到的是linux出了问题，查找各种日志未发现有何异常   
最后，终于找到问题的所在：python脚本使用统计时间的方式是time.clock()，而这种方式统计的是CPU的执行时间，不是程序的执行时间   
接下来，就几种python的统计时间方式对比一下：

**方法1：**  

	import datetime
	starttime = datetime.datetime.now()
	#long running
	#do something other
	endtime = datetime.datetime.now()
	print (endtime - starttime).seconds

**datetime.datetime.now()**获取的是当前日期，在程序执行结束之后，**这个方式获得的时间值为程序执行的时间**  

**方法2：**  

	start = time.time()
	#long running
	#do something other
	end = time.time()
	print end-start


**time.time()**获取自纪元以来的当前时间（以秒为单位）.如果系统时钟提供它们，则可能存在秒的分数.所以这个地方返回的是一个浮点型类型.这里获取的也是**程序的执行时间**  

**方法3：**  

	start = time.clock()
	#long running
	#do something other
	end = time.clock()
	print end-start


**time.clock()**返回程序开始或第一次被调用clock()以来的CPU时间.这具有与系统记录一样多的精度.返回的也是一个浮点类型.这里获得的是**CPU的执行时间**  
 
**`程序执行时间=cpu时间 + io时间 + 休眠或者等待时间`**

