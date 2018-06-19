---
layout:     post
title:      "Linux查找大文件和目录解析"
subtitle:   "find的使用说明"
date:       2018-06-19
author:     "kikyoar"
header-img: "img/post-bg-linux-version.jpg"
tags:
    - Linux
---  

## find的使用说明

**find命令用来在指定目录下查找文件。任何位于参数之前的字符串都将被视为欲查找的目录名。如果使用该命令时，不设置任何参数，则find命令将在当前目录下查找子目录与文件。并且将查找到的子目录和文件全部进行显示**  

**find(选项)(参数)** 

	-amin<分钟>：查找在指定时间曾被存取过的文件或目录，单位以分钟计算；
	-anewer<参考文件或目录>：查找其存取时间较指定文件或目录的存取时间更接近现在的文件或目录；
	-atime<24小时数>：查找在指定时间曾被存取过的文件或目录，单位以24小时计算；
	-cmin<分钟>：查找在指定时间之时被更改过的文件或目录；
	-cnewer<参考文件或目录>查找其更改时间较指定文件或目录的更改时间更接近现在的文件或目录；
	-ctime<24小时数>：查找在指定时间之时被更改的文件或目录，单位以24小时计算；
	-daystart：从本日开始计算时间；
	-depth：从指定目录下最深层的子目录开始查找；
	-expty：寻找文件大小为0 Byte的文件，或目录下没有任何子目录或文件的空目录；
	-exec<执行指令>：假设find指令的回传值为True，就执行该指令；
	-false：将find指令的回传值皆设为False；
	-fls<列表文件>：此参数的效果和指定“-ls”参数类似，但会把结果保存为指定的列表文件；
	-follow：排除符号连接；
	-fprint<列表文件>：此参数的效果和指定“-print”参数类似，但会把结果保存成指定的列表文件；
	-fprint0<列表文件>：此参数的效果和指定“-print0”参数类似，但会把结果保存成指定的列表文件；
	-fprintf<列表文件><输出格式>：此参数的效果和指定“-printf”参数类似，但会把结果保存成指定的列表文件；
	-fstype<文件系统类型>：只寻找该文件系统类型下的文件或目录；
	-gid<群组识别码>：查找符合指定之群组识别码的文件或目录；
	-group<群组名称>：查找符合指定之群组名称的文件或目录；
	-help或——help：在线帮助；
	-ilname<范本样式>：此参数的效果和指定“-lname”参数类似，但忽略字符大小写的差别；
	-iname<范本样式>：此参数的效果和指定“-name”参数类似，但忽略字符大小写的差别；
	-inum<inode编号>：查找符合指定的inode编号的文件或目录；
	-ipath<范本样式>：此参数的效果和指定“-path”参数类似，但忽略字符大小写的差别；
	-iregex<范本样式>：此参数的效果和指定“-regexe”参数类似，但忽略字符大小写的差别；
	-links<连接数目>：查找符合指定的硬连接数目的文件或目录；
	-iname<范本样式>：指定字符串作为寻找符号连接的范本样式；
	-ls：假设find指令的回传值为Ture，就将文件或目录名称列出到标准输出；
	-maxdepth<目录层级>：设置最大目录层级；
	-mindepth<目录层级>：设置最小目录层级；
	-mmin<分钟>：查找在指定时间曾被更改过的文件或目录，单位以分钟计算；
	-mount：此参数的效果和指定“-xdev”相同；
	-mtime<24小时数>：查找在指定时间曾被更改过的文件或目录，单位以24小时计算；
	-name<范本样式>：指定字符串作为寻找文件或目录的范本样式；
	-newer<参考文件或目录>：查找其更改时间较指定文件或目录的更改时间更接近现在的文件或目录；
	-nogroup：找出不属于本地主机群组识别码的文件或目录；
	-noleaf：不去考虑目录至少需拥有两个硬连接存在；
	-nouser：找出不属于本地主机用户识别码的文件或目录；
	-ok<执行指令>：此参数的效果和指定“-exec”类似，但在执行指令之前会先询问用户，若回答“y”或“Y”，则放弃执行命令；
	-path<范本样式>：指定字符串作为寻找目录的范本样式；
	-perm<权限数值>：查找符合指定的权限数值的文件或目录；
	-print：假设find指令的回传值为Ture，就将文件或目录名称列出到标准输出。格式为每列一个名称，每个名称前皆有“./”字符串；
	-print0：假设find指令的回传值为Ture，就将文件或目录名称列出到标准输出。格式为全部的名称皆在同一行；
	-printf<输出格式>：假设find指令的回传值为Ture，就将文件或目录名称列出到标准输出。格式可以自行指定；
	-prune：不寻找字符串作为寻找文件或目录的范本样式;
	-regex<范本样式>：指定字符串作为寻找文件或目录的范本样式；
	-size<文件大小>：查找符合指定的文件大小的文件；
	-true：将find指令的回传值皆设为True；
	-typ<文件类型>：只寻找符合指定的文件类型的文件；
	-uid<用户识别码>：查找符合指定的用户识别码的文件或目录；
	-used<日数>：查找文件或目录被更改之后在指定时间曾被存取过的文件或目录，单位以日计算；
	-user<拥有者名称>：查找符和指定的拥有者名称的文件或目录；
	-version或——version：显示版本信息；
	-xdev：将范围局限在先行的文件系统中；
	-xtype<文件类型>：此参数的效果和指定“-type”参数类似，差别在于它针对符号连接检查。   
	

## Linux查找大文件和目录解析

**df -h 查看硬盘所有挂载目录容量**

**du -sh 查看当前目录总共容量**

**du -h 查看当前目录文件夹所占容量（包含子目录）**

**du -sh /\*查询根目录各子目录所占容量（可改为想查询的目录）**  

要搜索当前目录下，超过800M大小的文件  

	find . -type f -size +800M  
	
详细显示一些文件属性或信息  

	find . -type f -size +800M  -print0 | xargs -0 ls -l  
	
只需要查找超过800M大小文件，并显示查找出来文件的具体大小

	find . -type f -size +800M  -print0 | xargs -0 du -h  
	
需要对查找结果按照文件大小做一个排序  

	find . -type f -size +800M  -print0 | xargs -0 du -h | sort -nr  

有时候搜索出来的结果太多了（譬如，我从根目录开始搜索），一直在刷屏，如果我只想查出最大的12个文件夹  

	du -hm --max-depth=2 | sort -nr | head -12  


## find命令中的print0和xargs -0  

**xargs命令的作用是将参数列表转换成小块分段传递给其他命令，以避免参数列表过长的问题**  

默认情况下, find 每输出一个文件名, 后面都会接着输出一个换行符 ('\n'), 因此我们看到的 find 的输出都是一行一行的:

	-(dearvoid@LinuxEden:Forum)-(~/tmp/find)-
	[bash-4.1.5] ; ls -l
	total 0
	-rw-r--r-- 1 root root 0 2010-08-02 18:09 file1.log
	-rw-r--r-- 1 root root 0 2010-08-02 18:09 file2.log
	-(dearvoid@LinuxEden:Forum)-(~/tmp/find)-
	[bash-4.1.5] ; find -name '*.log'
	./file2.log
	./file1.log
	-(dearvoid@LinuxEden:Forum)-(~/tmp/find)-
	[bash-4.1.5] ; bye

比如我想把所有的 .log 文件删掉, 可以这样配合 xargs 一起用:

	-(dearvoid@LinuxEden:Forum)-(~/tmp/find)-
	[bash-4.1.5] ; find -name '*.log'
	./file2.log
	./file1.log
	-(dearvoid@LinuxEden:Forum)-(~/tmp/find)-
	[bash-4.1.5] ; find -name '*.log' | xargs rm
	-(dearvoid@LinuxEden:Forum)-(~/tmp/find)-
	[bash-4.1.5] ; find -name '*.log'
	-(dearvoid@LinuxEden:Forum)-(~/tmp/find)-
	[bash-4.1.5] ; bye

嗯, 不错, find+xargs 真的很强大. 然而:

	-(dearvoid@LinuxEden:Forum)-(~/tmp/find)-
	[bash-4.1.5] ; ls -l
	total 0
	-rw-r--r-- 1 root root 0 2010-08-02 18:12 file 1.log
	-rw-r--r-- 1 root root 0 2010-08-02 18:12 file 2.log
	-(dearvoid@LinuxEden:Forum)-(~/tmp/find)-
	[bash-4.1.5] ; find -name '*.log'
	./file 1.log
	./file 2.log
	-(dearvoid@LinuxEden:Forum)-(~/tmp/find)-
	[bash-4.1.5] ; find -name '*.log' | xargs rm
	rm: cannot remove `./file': No such file or directory
	rm: cannot remove `1.log': No such file or directory
	rm: cannot remove `./file': No such file or directory
	rm: cannot remove `2.log': No such file or directory
	-(dearvoid@LinuxEden:Forum)-(~/tmp/find)-
	[bash-4.1.5] ; bye

原因其实很简单, xargs 默认是以空白字符 (空格, TAB, 换行符) 来分割记录的, 因此文件名 ./file 1.log 被解释成了两个记录 ./file 和 1.log, 不幸的是 rm 找不到这两个文件.

为了解决此类问题, 想出了一个办法, 让 find 在打印出一个文件名之后接着输出一个 NULL 字符 ('\0') 而不是换行符, 然后再告诉 xargs 也用 NULL 字符来作为记录的分隔符. 这就是 find 的 -print0 和 xargs 的 -0 的来历吧.

	-(dearvoid@LinuxEden:Forum)-(~/tmp/find)-
	[bash-4.1.5] ; ls -l
	total 0
	-rw-r--r-- 1 root root 0 2010-08-02 18:12 file 1.log
	-rw-r--r-- 1 root root 0 2010-08-02 18:12 file 2.log
	-(dearvoid@LinuxEden:Forum)-(~/tmp/find)-
	[bash-4.1.5] ; find -name '*.log' -print0 | hd
	           0  1  2  3   4  5  6  7   8  9  A  B   C  D  E  F  |0123456789ABCDEF|
	--------+--+--+--+--+---+--+--+--+---+--+--+--+---+--+--+--+--+----------------|
	00000000: 2e 2f 66 69  6c 65 20 31  2e 6c 6f 67  00 2e 2f 66  |./file 1.log../f|
	00000010: 69 6c 65 20  32 2e 6c 6f  67 00                     |ile 2.log.      |
	-(dearvoid@LinuxEden:Forum)-(~/tmp/find)-
	[bash-4.1.5] ; find -name '*.log' -print0 | xargs -0 rm
	-(dearvoid@LinuxEden:Forum)-(~/tmp/find)-
	[bash-4.1.5] ; find -name '*.log'
	-(dearvoid@LinuxEden:Forum)-(~/tmp/find)-
	[bash-4.1.5] ; bye

为什么要选 '\0' 而不是其他字符做分隔符呢? 这个也容易理解: 一般的编程语言中都用 '\0' 来作为字符串的结束标志, 文件的路径名中不可能包含 '\0' 字符.

为什么不用-exec，而用xargs ?因为find会把找到的记录都给后面的命令传过去执行，-exec有长度限制，可能会出现参数溢出。

