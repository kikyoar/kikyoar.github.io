---
layout:     post
title:      "Python \_\_getattribute\_\_ VS \_\_getattr\_\_ 浅谈"
subtitle:   "\_\_getattr\_\_和\_\_getattribute\_\_函数的使用区别"
date:       2019-03-12
author:     "kikyoar"
header-img: "img/post-bg-python-version.jp"
tags:
    - Python
---  

[摘自简书](https://www.jianshu.com/p/885d59db57fc)

**getattr()函数**   

- getattr()函数是普通函数，它和特殊函数\_\_getattr\_\_()不是一回事
- getattr()函数会在你试图读取一个不存在的属性时，引发AttributeError异常  

**\_\_getattr\_\_()函数**  

- \_\_getattr\_\_()函数是特殊函数
- 仅当属性不能在实例的\_\_dict\_\_或它的类，或者祖先类中找到时，才被调用  
- 主要用\_\_getattr\_\_()来实现类的灵活性，或者用它来做一些兜底的操作  

**\_\_getattribute\_\_()函数**  

- \_\_getattribute\_\_() 在查找真正要访问的属性之前就被调用了，无论该属性是否存在  
- 使用\_\_getattribute\_\_()要特别注意，**因为如果你在它里面不知何故再次调用了\_\_getattribute\_\_()，你就会进入无穷递归**。为了避免这种情况，如果你想要访问任何它需要的属性，应该总是调用祖先类的同名方法：比如super(obj, self).\_\_getattribute\_\_(attr)
- 主要使用\_\_getattribute\_\_()来实现某些拦截功能，特别是数据访问保护  


### 举个栗子

**用作实例属性的获取和拦截**  

当访问某个实例属性时， getattribute会被无条件调用，如未实现自己的getattr方法，会抛出AttributeError提示找不到这个属性，如果自定义了自己getattr方法的话，方法会在这种找不到属性的情况下被调用，所以在找不到属性的情况下通过实现自定义的getattr方法来实现一些功能是一个不错的方式，因为它不会像getattribute方法每次都会调用可能会影响一些正常情况下的属性访问：

	class Test(object):
	    def __init__(self, p):
	        self.p = p
	
	    def __getattr__(self, item):
	        return 'default'
	
	t = Test('p1')
	print t.p
	print t.p2
	
	>>> p1
	>>> default

**自定义getattribute的时候防止无限递归**  

因为getattribute在访问属性的时候一直会被调用，自定义的getattribute方法里面同时需要返回相应的属性，通过self.\_\_dict\_\_取值会继续向下调用getattribute，造成循环调用：

	class AboutAttr(object):
	    def __init__(self, name):
	        self.name = name
	
	    def __getattribute__(self, item):
	        try:
	            return super(AboutAttr, self).__getattribute__(item)
	        except KeyError:
	            return 'default'

这里通过调用绑定的super对象来获取队形的属性，对新式类来说其实和object.\_\_getattribute\_\_(self, item)一样的道理:

- 默认情况下自定义的类会从object继承getattribute方法，对于属性的查找是完全能用的
- getattribute的实现感觉还是挺抽象化的，只需要绑定相应的实例对象和要查找的属性名称就行

**同时覆盖掉getattribute和getattr的时候，在getattribute中需要模仿原本的行为抛出AttributeError或者手动调用getattr**  

	class AboutAttr(object):
	    def __init__(self, name):
	        self.name = name
	
	    def __getattribute__(self, item):
	        try:
	            return super(AboutAttr, self).__getattribute__(item)
	        except KeyError:
	            return 'default'
	        except AttributeError as ex:
	            print ex
	
	    def __getattr__(self, item):
	        return 'default'
	
	at = AboutAttr('test')
	print at.name
	print at.not_exised
	
	>>>test
	>>>'AboutAttr' object has no attribute 'not_exised'
	>>>None

上面例子里面的getattr方法根本不会被调用，因为原本的AttributeError被我们自行处理并未抛出，也没有手动调用getattr，所以访问not_existed的结果是None而不是default
