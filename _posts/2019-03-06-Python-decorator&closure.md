---
layout:     post
title:      "Python装饰器详解"
subtitle:   "闭包和装饰器"
date:       2019-03-06
author:     "kikyoar"
header-img: "img/post-bg-python-version.jp"
tags:
    - Python
--- 

[摘自简书](https://www.jianshu.com/p/bbbac606697f)

## 闭包

在函数编程中经常用到闭包。闭包是什么，它是怎么产生的及用来解决什么问题呢。给出字面的定义先：闭包是由函数及其相关的引用环境组合而成的实体(即：**闭包=函数+引用环境**)。这个从字面上很难理解，特别对于一直使用命令式语言进行编程的程序员们。**要形成闭包，首先得有一个嵌套的函数，即函数中定义了另一个函数，闭包则是一个集合，它包括了外部函数的局部变量，这些局部变量在外部函数返回后也继续存在，并能被内部函数引用**  

`举个例子`  

这是个经常使用到的例子，定义一个函数generate\_power\_func，它返回另一个函数，现在闭包形成的条件已经达到  

	def generate_power_func(n):
	    print "id(n): %X" % id(n)
	    def nth_power(x):
	        return x**n
	    print "id(nth_power): %X" % id(nth_power)
	    return nth_power  
	    
	    
对于内部函数nth_power，它能引用到外部函数的局部变量n，而且即使generate\_power\_func已经返回。把这种现象就称为闭包。具体使用一下  

	>>> raised_to_4 = generate_power_func(4)
	id(n): 246F770
	id(nth_power): 2C090C8
	>>> repr(raised_to_4)
	'<function nth_power at 0x2c090c8>'  
	
从结果可以看出，当generate\_power\_func(4)执行后, 创建和返回了nth\_power这个函数对象，内存地址是 0x2C090C8,并且发现raised\_to\_4和它的内存地址相同，即raised\_to\_4只是这个函数对象的一个引用  


## 装饰器  

装饰器本质上是一个 Python 函数／类，它可以让其他函数／类在**不需要做任何代码修改的前提下增加额外功能**，**装饰器的返回值也是一个函数/类对象**。它经常用于有切面需求的场景，比如：插入日志、性能测试、权限校验等场景，装饰器是解决这类问题的绝佳设计。有了装饰器，我们就可以抽离出与函数功能本身无关的代码到装饰器中复用。概括的讲，**装饰器的作用就是为已经存在的对象添加额外的功能**  

### 简单装饰器（不使用@符号）  

下面是一个简单的装饰器例子，log\_append 来装饰 direct\_print，这种写法支持任意参数的direct_print，也支持任意参数的装饰函数log\_append  
	
	def log_append():
	    def decorator(func):
	        def wrapper(**kwargs):
	            if "name" in kwargs:
	                print("check name %s" % (kwargs.get("name")))
	            else:
	                print("check name fail" )
	                return
	            return func(**kwargs)
	        return wrapper
	    return decorator
	
	def direct_print(**kwargs):
	    print("print name is %s"% (kwargs.get("name")))
	
	    
	fun_decoratored=log_append()(direct_print)
	fun_decoratored(names="111")
	
	fun_decoratored(name="111")
	
	print(fun_decoratored)
	
	direct_print(name="111")

结果为：

	check name fail
	check name 111
	print name is 111
	<function log_append.<locals>.decorator.<locals>.wrapper at 0x100bb11e0>
	print name is 111  
	
	
### 用@符号
利用@符号，可以提高代码的阅读性,我们用@的方式改写上面的代码
	
	def log_append():
	    def decorator(func):
	        def wrapper(**kwargs):
	            if "name" in kwargs:
	                print("check name %s" % (kwargs.get("name")))
	            else:
	                print("check name fail" )
	                return
	            return func(**kwargs)
	        return wrapper
	    return decorator
	@log_append()
	def direct_print(**kwargs):
	    print("print name is %s"% (kwargs.get("name")))	    	
	
	direct_print(name="111")
	
	direct_print(names="111")


### 装饰器函数传递参数并体现装饰器便利性  

假如现在接口强制校验的规则不定，一个函数强制校验name，另一个强制校验names字段，后面可能还会强制校验key等别的字段，这时候改写log\_append就比较方便了  

下面的代码在完全不改变direct\_print 业务逻辑的情况下，重写了校验逻辑，并且复用在direct\_print\_names 方法  


	from collections import Iterable
	
	
	def log_append(au_keys=[]):
	    def decorator(func):
	        def wrapper(**kwargs):
	            if not isinstance(au_keys,Iterable):
	                raise Exception("error au_keys,should be Iterable str")
	            for check_key in au_keys:
	                if check_key in kwargs:
	                    print("check %s %s" % (check_key,kwargs.get(check_key)))
	                else:
	                    print("check %s fail\n"% (check_key))
	                    return
	            return func(**kwargs)
	        return wrapper
	    return decorator
	@log_append(["name"])
	def direct_print(**kwargs):
	    print("print name is %s\n"% (kwargs.get("name")))
	
	@log_append(["names"])
	def direct_print_names(**kwargs):
	    print("print names is %s\n"% (kwargs.get("names")))
	
	
	direct_print(name="111")
	
	direct_print(names="111")
	
	
	direct_print_names(name="111")
	
	direct_print_names(names="111")  
	
	
### functools.wraps 保留函数信息  

不使用functools.wraps会导致 docstring、name这些函数元信息丢失，比如 使用 direct\_print\_names.name 会返回wrapper，这样很影响使用，所以我们需要functools.wraps。加入方式简单，再装饰函数传入func的地方加入@wraps  

	from collections import Iterable
	from functools import wraps
	
	
	def log_append(au_keys=[]):
	    def decorator(func):
	        @wraps(func)
	        def wrapper(**kwargs):
	            print()
	            if not isinstance(au_keys,Iterable):
	                raise Exception("error au_keys,should be Iterable str")
	            for check_key in au_keys:
	                if check_key in kwargs:
	                    print("check %s %s" % (check_key,kwargs.get(check_key)))
	                else:
	                    print("check %s fail\n"% (check_key))
	                    return
	            return func(**kwargs)
	        return wrapper
	    return decorator
	@log_append(["name"])
	def direct_print(**kwargs):
	    print("print name is %s\n"% (kwargs.get("name")))
	
	@log_append(["names"])
	def direct_print_names(**kwargs):
	    print("print names is %s\n"% (kwargs.get("names")))
	
	
	direct_print(name="111")
	
	direct_print(names="111")
	
	
	direct_print_names(name="111")
	
	direct_print_names(names="111")
	
	print(direct_print.__name__)


### 调用顺序  

python支持多重装饰的，使用方式就是一个一个的@写下去，它的执行顺序是从外到里，最先调用最外层的装饰器，最后调用最里层的装饰器，在上面的代码里面加入打印时间的装饰函数print\_time,让direct\_print 先装饰print\_time，direct\_print\_names后装饰，实际上 direct\_print 效果等于`print_time(log_append(["name"])(direct_print))`，direct\_print\_names效果等于`log_append(["_names"])(print_time(direct_print))`  

	from collections import Iterable
	from functools import wraps
	
	import time
	
	
	def log_append(au_keys=[]):
	    def decorator(func):
	        @wraps(func)
	        def wrapper(**kwargs):
	            print()
	            if not isinstance(au_keys,Iterable):
	                raise Exception("error au_keys,should be Iterable str")
	            for check_key in au_keys:
	                if check_key in kwargs:
	                    print("check %s %s" % (check_key,kwargs.get(check_key)))
	                else:
	                    print("check %s fail\n"% (check_key))
	                    return
	            return func(**kwargs)
	        return wrapper
	    return decorator
	
	def print_time(func):
	    @wraps(func)
	    def wrapper(**kwargs):
	        print("time is %s"%(time.strftime("%Y-%m-%d %H:%M;%S",time.localtime())))
	        return func(**kwargs)
	    return wrapper
	
	@print_time
	@log_append(["name"])
	def direct_print(**kwargs):
	    print("print name is %s\n"% (kwargs.get("name")))
	
	@log_append(["names"])
	@print_time
	def direct_print_names(**kwargs):
	    print("print names is %s\n"% (kwargs.get("names")))
	
	
	direct_print(name="111")
	
	direct_print(names="111")
	
	
	direct_print_names(name="111")
	
	direct_print_names(names="111")
	
	print(direct_print.__name__)  
	
	


# 小结


- **1、装饰器原理** 

		def w1(func):
		    def inner():
		        print('---正在验证权限----')
		        func()
		    return inner
		def f1():
		    print('-----f1-----')
		
		def f2():
		    print('----f2-----')

		#innerfunc = w1(f1)
		#innerfunc()
		
		f1 = w1(f1)
		f1()      

- **2、装饰器语法**  

		def w1(func):
		    def inner():
		        print('---正在验证权限----')
		        func()
		    return inner
		
		@w1 #在func代表的函数身上添加@
		def f1():
		    print('-----f1-----')
		
		def f2():
		    print('----f2-----')
		
		f1()

- **3、装饰器执行的时间**  

	**装饰器在Python解释器执行的时候，就会进行自动装饰，并不是在在执行的时候才进行装饰**  

- **4、两个装饰器**  

		def w1(func):
		    print('---正在装饰 1----')
		    def inner():
		        print('---正在验证权限 1----')
		        func()
		    return inner
		
		def w2(func):
		    print('---正在装饰 2----')
		    def inner():
		        print('---正在验证权限 2----')
		        func()
		    return inner
		
		@w1 #f1 = w1(f1)
		@w2 #f1 = w2(f1)
		def f1():
		    print('-----f1-----')

		#在调用f1之前，已经进行了装饰
		f1()

	说明:如果不调用f1，直接编译，打印结果是：
	
		---正在装饰 2----
		---正在装饰 1----

	如果调用f1，打印结果是
	
		---正在装饰 2----
		---正在装饰 1----
		---正在验证权限 1----
		---正在验证权限 2----
		-----f1-----

	理解：  
	 
		@w1 #f1 = w1(f1)
		@w2 #f1 = w2(f1)
		def f1():


	当解释器程序执行这个的时候，先碰到@w1所以判断下面是否是def函数，发现下面是@w2，所以继续执行，当碰到def f1()的时候，找到要装饰的目标函数，所以先用w2对目标进行装饰，然后用w1对目标函数进行装饰

- **5、装饰器对有参和无参的函数进行装饰**  
	1、有参
	
		def func(funNume):
		    def func_in(aa,bb):
		        funNume(aa,bb)
		    return  func_in
		
		@func
		def f1(a,b):
		    print("------ppppp-----")
		    
		f1(11,12)
	
	2、对有返回值的函数进行装饰
	
		def func(funNume):
		    def func_in(aa,bb):
		        return funNume(aa,bb)
		
		    return  func_in
		
		@func
		def f1(a,b):
		    print("------ppppp-----")
		    return ‘haha’
		    
		f1(11,12)

- **6 、通用装饰器**  

		def func(funNume):
		    def func_in(*args,**kargs):
		        return funNume(*args,**kargs)
		    return  func_in
		
		@func
		def f1(a,b):
		    print("------f1-----")
		    return 'haha'
		
		@func
		def f2():
		    print("----f2-----")
		
		@func
		def f3(a):
		    print("----f3-----")
		#说明：*args,**kargs是不定参数

- **7、带参数的装饰器**  

		def func_arg(arg = 'hello'):
		    def func(funNume):
		        def func_in(*args,**kargs):
		            return funNume(*args,**kargs)
		        return  func_in
		    return func
		
		#1.先执行func_arg('world')函数，这个函数return的结果是func这个函数的引用
		#2.@func
		#3.使用@func对f1进行装饰
		@func_arg('world')
		def f1(a,b):
		    print("------f1-----")
		    return 'haha'

