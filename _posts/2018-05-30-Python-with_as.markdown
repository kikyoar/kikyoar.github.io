---
layout:     post
title:      "with ... as 语句的来龙去脉"
subtitle:   "With ... as 语法解释"
date:       2018-05-30
author:     "kikyoar"
header-img: "img/post-bg-python-version.jpg"
tags:
    - Python
---   

**示例 1**  

	with open('courses.txt') as f:
	    for i in f:
	        print(i.strip())   

打开一个文件，然后循环做一些事情。但是你知道为什么会有 with 吗？我们自己是不是能够写出可以作用在 with 关键字上的对象呢？  

现在，我们带着上述两个问题来说一说 with 的由来以及上下文管理器相关内容。  

with 语句的目的是简化 try/finally 模式。这种模式用于保证一段代码运行完毕后执行某项操作，即便那段代码由于异常、return 语句或sys.exit() 调用而中止，也会执行指定的操作。finally 子句中的代码通常用于释放重要的资源，或者还原临时变更的状态。  

示例1的功能我们可以使用 try/finally 模式实现：  

**示例 2**   

	try:
	    f = open('courses.txt')
	    for i in f:
	        print(i.strip())
	finally:
	    f.close()  
	    
try中的 except 和 else 不是必须的，这里为了简单明了，我们只用了 finally。

对比两个示例，我们可以看到示例1相对简洁，这就是 with 的由来。

其实，语言中的一些特性或者说一些亮眼的特性，必然是有一个演化的过程的，我们作为后来者和使用者应该多花一些心思去思索其背后的实现过程，相信你会收获更多。

那么 上下文管理器 又是什么呢？

上下文管理器协议包含 \__enter\__ 和 \__exit\__ 两个方法。with 语句开始运行时，会在上下文管理器对象上调用 \__enter\__ 方法。with 语句运行结束后，会在上下文管理器对象上调用 \__exit\__ 方法，以此扮演 finally 子句的角色。最常见的例子是确保关闭文件对象（示例1）。

下面我们实现一个简单的例子来直观的感受一下：  

**示例 3**   

	class T:
	    def __enter__(self):
	        print('T.__enter__')
	        return '我是__enter__的返回值'
	    def __exit__(self, exc_type, exc_val, exc_tb):
	        print('T.__exit__')
	
	with T() as t:
	    print(t)

输出：

	T.__enter__
	我是__enter__的返回值
	T.__exit__  

示例3中实现了一个类T，它的对象包含了\__enter\__和\__exit\__方法，有了这两个方法就可以使用 with 处理该对象。执行 with 后面的表达式T()得到的是上下文管理器对象，通过as字句把对象绑定到了变量t上。

观察输出结果，可以看到with块先调用了\__enter\__方法，在处理完内部逻辑（print(t)）之后调用了exit方法，而t其实就是\__enter\__方法的返回值。

当然，这个例子只是为了方便我们理解上下文管理器，下面我们看一个更有意思的例子：  

**示例 4**  

	obj1 = HaHa('你手机拿反了')
	with obj1 as content:
	    print('哈哈镜花缘')
	    print(content)
	print('#### with 执行完毕后，再输出content: ####')
	print(content)  

输出：

	缘花镜哈哈
	了反拿机手你
	#### with 执行完毕后，在输出content: ####
	你手机拿反了  

示例4中，上下文管理器是 HaHa 类的实例，Python 调用此实例的 \__enter\__ 方法，把返回结果绑定到 变量content 上。

打印一个字符串，然后打印 content 变量的值。可以看到打印出的内容都是是反向的。

最后，当 with 块已经执行完毕。可以看出，\__enter\__ 方法返回的值——即存储在 content 变量中的值——是字符串 ‘你手机拿反了’。输出不再是反向的了。  

**HaHa类的实现：**   

	import sys
	
	class HaHa:
	
	    def __init__(self, word):
	        self.word = word
	
	    def reverse_write(self, text):
	        self.original_write(text[::-1])
	
	    def __enter__(self):
	        self.original_write = sys.stdout.write
	        sys.stdout.write = self.reverse_write
	        return self.word
	
	    def __exit__(self, exc_type, exc_value, traceback):
	        sys.stdout.write = self.original_write
	        return True   

在\__enter\__方法中，我们接管了标准输出，将其替换成我们自己编写的方法reverse_write，reverse_write方法将参数内容反转。而在\__exit\__方法中，我们将标准输出还原。\__exit\__方法需要返回True。

总之，with之于上下文管理器，就像for之于迭代器一样。with就是为了方便上下文管理器的使用。

上下文管理器特性在标准库中有一些应用：   
	
- 在 sqlite3 模块中用于管理事务;  
- 在 threading 模块中用于维护锁、条件和信号;  
另外，说到上下文管理器就不得不提一下@contextmanager 装饰器，它能减少创建上下文管理器的样板代码量，因为不用编写一个完整的类，定义 \__enter\__和 \__exit\__ 方法，而只需实现有一个 yield 语句的生成器，生成想让 \__enter\__ 方法返回的值。

在使用 @contextmanager 装饰的生成器中，yield 语句的作用是把函数的定义体分成两部分：  

- yield 语句前面的所有代码在 with 块开始时（即解释器调用 \__enter\__ 方法时）执行

- yield 语句后面的代码在with 块结束时（即调用 \__exit\__ 方法时）执行。

下面我们用 @contextmanager 装饰器来实现一下示例4的功能：  

**示例 5**   

	import sys
	import contextlib
	
	@contextlib.contextmanager
	def WoHa(n):
	    original_write = sys.stdout.write
	    def reverse_write(text):
	        original_write(text[::-1])
	    sys.stdout.write = reverse_write
	    yield n
	    sys.stdout.write =  original_write
	    return True
	
	obj1 = WoHa('你手机拿反了')
	with obj1 as content:
	    print('哈哈镜花缘')
	    print(content)
	print('#### with 执行完毕后，在输出content: ####')
	print(content)    
	
输出：  

	缘花镜哈哈
	了反拿机手你
	#### with 执行完毕后，在输出content: ####
	你手机拿反了  
	
这里我们需要注意的是：代码执行到yield时，会产出一个值，这个值会绑定到 with 语句中 as 子句的变量上。执行 with 块中的代码时，这个函数会在yield这里暂停。此时，相当于示例4中执行完\__enter\__方法。而控制权一旦跳出 with 块（块内代码执行完毕）则继续执行 yield 语句之后的代码。

@contextmanager 装饰器优雅且实用，把三个不同的 Python 特性结合到了一起：函数装饰器、生成器和 with 语句。



 

