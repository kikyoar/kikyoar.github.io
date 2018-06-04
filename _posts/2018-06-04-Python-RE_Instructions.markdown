---
layout:     post
title:      "Python 正则表达式用法"
subtitle:   "正则表达式和python re模块"
date:       2018-06-04
author:     "kikyoar"
header-img: "img/post-bg-python-version.jpg"
tags:
    - Linux
---   

## 基础语法

**正则表达式本身是一种小型的、高度专业化的编程语言，而在python中，通过内嵌集成re模块，程序员们可以直接调用来实现正则匹配。正则表达式模式被编译成一系列的字节码，然后由用C编写的匹配引擎执行**    

**1、普通字符和11个元字符：**   

字符 | 描述  | 列举  |  结果
:--------:|:--------|:--------:|:-------:  
普通字符|	匹配自身  | abc | abc  
.|匹配任意除换行符"\n"外的字符(在DOTALL模式中也能匹配换行符|a.c|abc
\*|匹配前一个字符0或多次|abc*|ab;   abccc
+|匹配前一个字符1次或无限次|abc+|abc;  abccc
?|匹配一个字符0次或1次|abc?|ab;  abc
^|匹配字符串开头;在多行模式中匹配每一行的开头|\^abc|abc
\$|匹配字符串末尾，在多行模式中匹配每一行的末尾|abc\$|abc
\|	|或。匹配\|左右表达式任意一个，从左到右匹配，如果\|没有包括在()中，则它的范围是整个正则表达式|abc\|def|abc def
{}|{m}匹配前一个字符m次，{m,n}匹配前一个字符m至n次，若省略n，则匹配m至无限次|ab{1,2}c|abc  abbc
[]|字符集。对应的位置可以是字符集中任意字符。字符集中的字符可以逐个列出，也可以给出范围，如[abc]或[a-c]。[\^abc]表示取反，即非abc。所有特殊字符在字符集中都失去其原有的特殊含义。用\\反斜杠转义恢复特殊字符的特殊含义| a[bcd]e| abe  ace  ade
()| 被括起来的表达式将作为分组，从表达式左边开始没遇到一个分组的左括号“（”，编号+1.分组表达式作为一个整体，可以后接数量词。表达式中的\|仅在该组中有效。|(abc){2}a(123\|456)c|abcabc  a456c   

**这里需要强调一下反斜杠\的作用：**  

- 反斜杠后边跟元字符去除特殊功能；（即将特殊字符转义成普通字符）
- 反斜杠后边跟普通字符实现特殊功能；（即预定义字符）
- 引用序号对应的字组所匹配的字符串   

以下是代码：  

	a=re.search(r'(tina)(fei)haha\2','tinafeihahafei tinafeihahatina').group()
	print(a)
	结果：
	tinafeihahafei  
	


**2、预定义字符集（可以写在字符集[...]中）**   

字符 | 描述  | 列举  |  结果  
:--------:|:--------|:--------:|:-------:  
\d|数字:[0-9]|a\bc|a1c  
\D|非数字:[\^\d]|a\Dc|abc
\s|匹配任何空白字符:[<空格>\t\r\n\f\v]|a\sc|a c
\S|非空白字符:[\^\s]|a\Sc|abc
\w|匹配包括下划线在内的任何字字符:[A-Za-z0-9_]|a\wc|abc
\W|匹配非字母字符，即匹配特殊字符|a\Wc|a c
\A|仅匹配字符串开头,同\^|\Aabc|abc
\Z|仅匹配字符串结尾，同$|abc\Z|abc
\b|匹配\w和\W之间，即匹配单词边界匹配一个单词边界，也就是指单词和空格间的位置。例如， 'er\b' 可以匹配"never" 中的 'er'，但不能匹配 "verb" 中的 'er'|\babc\ba\b!bc| abc  a!bc
\B|[\^\b]|a\Bbc|abc  

	这里需要强调一下\b的单词边界的理解：
	w = re.findall('\btina','tian tinaaaa')
	print(w)
	s = re.findall(r'\btina','tian tinaaaa')
	print(s)
	v = re.findall(r'\btina','tian#tinaaaa')
	print(v)
	a = re.findall(r'\btina\b','tian#tina@aaa')
	print(a)
	执行结果如下：
	[]
	['tina']
	['tina']
	['tina']  
	


**3、特殊分组用法：**   

字符 | 描述  | 列举  |  结果  
:--------:|:--------|:--------:|:-------:   
(?P\<name\>)|分组，除了原有的编号外再指定一个额外的别名|(?P\<id\>abc){2}|abcabc
(?P=name)|引用别名为<name>的分组匹配到字符串|(?P<id>\d)abc(?P=id)|1abc1 5abc5
\<number\>|引用编号为<number>的分组匹配到字符串|(\d)abc\1|1abc1 5abc5   


## Re模块的方法：  

**1、compile()**  

编译正则表达式模式，返回一个对象的模式。（可以把那些常用的正则表达式编译成正则表达式对象，这样可以提高一点效率。）

**格式：**

	re.compile(pattern,flags=0)

- pattern: 编译时用的表达式字符串。
- flags 编译标志位，用于修改正则表达式的匹配方式，如：是否区分大小写，多行匹配等。常用的flags有：  

	标志 | 含义 
	:--------:|:--------:    
	re.S(DOTALL)|使.匹配包括换行在内的所有字符  
	re.I（IGNORECASE）|使匹配对大小写不敏感
	re.L（LOCALE）|做本地化识别（locale-aware)匹配，法语等
	re.M(MULTILINE)|多行匹配，影响\^和\$
	re.X(VERBOSE)|该标志通过给予更灵活的格式以便将正则表达式写得更易于理解
	re.U|根据Unicode字符集解析字符，这个标志影响\w,\W,\b,\B

**参考代码：**  

	import re
	tt = "Tina is a good girl, she is cool, clever, and so on..."
	rr = re.compile(r'\w*oo\w*')
	print(rr.findall(tt))   #查找所有包含'oo'的单词
	执行结果如下：
	['good', 'cool']

**2、match()**  

决定RE是否在字符串刚开始的位置匹配。//**注：这个方法并不是完全匹配。当pattern结束时若string还有剩余字符，仍然视为成功。想要完全匹配，可以在表达式末尾加上边界匹配符'$'**

格式：

	re.match(pattern, string, flags=0)

**参考代码：** 

	print(re.match('com','comwww.runcomoob').group())
	print(re.match('com','Comwww.runcomoob',re.I).group())
	执行结果如下：
	com
	com  
	
**3、search()**  

格式：

	re.search(pattern, string, flags=0)

re.search函数会在字符串内查找模式匹配,只要找到第一个匹配然后返回，如果字符串没有匹配，则返回None  

	print(re.search('\dcom','www.4comrunoob.5com').group())
	执行结果如下：
	4com
**注：match和search一旦匹配成功，就是一个match object对象，而match object对象有以下方法：**

- start() 返回匹配开始的位置
- end() 返回匹配结束的位置
- span() 返回一个元组包含匹配 (开始,结束) 的位置
- group() 返回re整体匹配的字符串，可以一次输入多个组号，对应组号匹配的字符串  

a. group（）:返回re整体匹配的字符串  
b. group (n,m) :返回组号为n，m所匹配的字符串，如果组号不存在，则返回indexError异常  
c.groups（）:groups() 方法返回一个包含正则表达式中所有小组字符串的元组，从 1 到所含的小组号，通常groups()不需要参数，返回一个元组，元组中的元就是正则表达式中定义的组   

**参考代码：**   

	import re
	a = "123abc456"
	print(re.search("([0-9]*)([a-z]*)([0-9]*)",a).group(0))   #123abc456,返回整体
	print(re.search("([0-9]*)([a-z]*)([0-9]*)",a).group(1))   #123
	print(re.search("([0-9]*)([a-z]*)([0-9]*)",a).group(2))   #abc
	print(re.search("([0-9]*)([a-z]*)([0-9]*)",a).group(3))   #456

group(1) 列出第一个括号匹配部分，group(2) 列出第二个括号匹配部分，group(3) 列出第三个括号匹配部分  

**4、findall()**  

re.findall遍历匹配，可以获取字符串中所有匹配的字符串，返回一个列表  

格式：

	re.findall(pattern, string, flags=0)  

**参考代码1：**   

	p = re.compile(r'\d+')
	print(p.findall('o1n2m3k4'))
	执行结果如下：
	['1', '2', '3', '4']  

**参考代码2：**   

	import re
	tt = "Tina is a good girl, she is cool, clever, and so on..."
	rr = re.compile(r'\w*oo\w*')
	print(rr.findall(tt))
	print(re.findall(r'(\w)*oo(\w)',tt))         #()表示子表达式 
	执行结果如下：
	['good', 'cool']
	[('g', 'd'), ('c', 'l')]  
	
**5、finditer()**  

搜索string，返回一个顺序访问每一个匹配结果（Match对象）的迭代器。找到 RE 匹配的所有子串，并把它们作为一个迭代器返回

格式：

	re.finditer(pattern, string, flags=0)  

**参考代码：**

	iter = re.finditer(r'\d+','12 drumm44ers drumming, 11 ... 10 ...')
	for i in iter:
	    print(i)
	    print(i.group())
	    print(i.span())
	
	
	执行结果如下：
	<_sre.SRE_Match object; span=(0, 2), match='12'>
	(0, 2)
	<_sre.SRE_Match object; span=(8, 10), match='44'>
	(8, 10)
	<_sre.SRE_Match object; span=(24, 26), match='11'>
	(24, 26)
	<_sre.SRE_Match object; span=(31, 33), match='10'>
	(31, 33)

**6、split()**  

按照能够匹配的子串将string分割后返回列表  

可以使用re.split来分割字符串，如：re.split(r'\s+', text)；将字符串按空格分割成一个单词列表  

格式：

	re.split(pattern, string[, maxsplit])

maxsplit用于指定最大分割次数，不指定将全部分割  

**参考代码：**  

	print(re.split('\d+','one1two2three3four4five5'))
	执行结果如下：
	['one', 'two', 'three', 'four', 'five', '']

**7、sub()**   

使用re替换string中每一个匹配的子串后返回替换后的字符串

格式：

	re.sub(pattern, repl, string, count)  

**参考代码：**	

	import re
	text = "JGood is a handsome boy, he is cool, clever, and so on..."
	print(re.sub(r'\s+', '-', text))
	
	执行结果如下：
	JGood-is-a-handsome-boy,-he-is-cool,-clever,-and-so-on...

其中第二个函数是替换后的字符串；本例中为'-'  
第四个参数指替换个数。默认为0，表示每个匹配项都替换  

re.sub还允许使用函数对匹配项的替换进行复杂的处理。

如：re.sub(r'\s', lambda m: '[' + m.group(0) + ']', text, 0)；将字符串中的空格' '替换为'[ ]'  

	import re
	text = "JGood is a handsome boy, he is cool, clever, and so on..."
	print(re.sub(r'\s+', lambda m:'['+m.group(0)+']', text,0))
	
	执行结果如下：
	JGood[ ]is[ ]a[ ]handsome[ ]boy,[ ]he[ ]is[ ]cool,[ ]clever,[ ]and[ ]so[ ]on...  
	
**8、subn()**  

返回替换次数

格式：

	subn(pattern, repl, string, count=0, flags=0)  

**参考代码：**		

	print(re.subn('[1-2]','A','123456abcdef'))
	print(re.sub("g.t","have",'I get A,  I got B ,I gut C'))
	print(re.subn("g.t","have",'I get A,  I got B ,I gut C'))
	执行结果如下：
	('AA3456abcdef', 2)
	I have A,  I have B ,I have C
	('I have A,  I have B ,I have C', 3)  
	

## 一些注意点  

**1、re.match与re.search与re.findall的区别：**  

re.match只匹配字符串的开始，如果字符串开始不符合正则表达式，则匹配失败，函数返回None；而re.search匹配整个字符串，直到找到一个匹配  

	a=re.search('[\d]',"abc33").group()
	print(a)
	p=re.match('[\d]',"abc33")
	print(p)
	b=re.findall('[\d]',"abc33")
	print(b)
	
	执行结果：
	3
	None
	['3', '3']  
	
**2、贪婪匹配与非贪婪匹配：**  

\*?,+?,??,{m,n}?    前面的\*,+,?等都是贪婪匹配，也就是尽可能匹配，后面加?号使其变成惰性匹配

**参考代码1：**	

	a = re.findall(r"a(\d+?)",'a23b')
	print(a)
	b = re.findall(r"a(\d+)",'a23b')
	print(b)
	执行结果：
	['2']
	['23']
	
	a = re.match('<(.*)>','<H1>title<H1>').group()
	print(a)
	b = re.match('<(.*?)>','<H1>title<H1>').group()
	print(b)
	执行结果：
	<H1>title<H1>
	<H1>

**参考代码2：**	
	
	a = re.findall(r"a(\d+)b",'a3333b')
	print(a)
	b = re.findall(r"a(\d+?)b",'a3333b')
	print(b)
	执行结果如下：
	['3333']
	['3333']

这里需要注意的是如果前后均有限定条件的时候，就不存在什么贪婪模式了，非匹配模式失效    

**3、用flags时遇到的小坑**

	print(re.split('a','1A1a2A3',re.I))  #输出结果并未能区分大小写  
	
这是因为re.split(pattern，string，maxsplit,flags)默认是四个参数，当我们传入的三个参数的时候，系统会默认re.I是第三个参数，所以就没起作用。如果想让这里的re.I起作用，写成flags=re.I即可







