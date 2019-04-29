---
layout:     post
title:      "tar 实现 Linux 操作系统备份"
subtitle:   "tar软件包管理"
date:       2019-04-29
author:     "kikyoar"
header-img: "img/post-bg-linux-version.jpg"
tags:
    - Linux
---  

[摘自Linux企业运维实战]()


### tar 命令参数详解  

	-A或--catenate：新增文件到以存在的备份文件；
	
	-B：设置区块大小；
	
	-c或--create：建立新的备份文件；
	
	-C <目录>：这个选项用在解压缩，若要在特定目录解压缩，可以使用这个选项。
	
	-d：记录文件的差别；
	
	-x或--extract或--get：从备份文件中还原文件；
	
	-t或--list：列出备份文件的内容；
	
	-z或--gzip或--ungzip：通过gzip指令处理备份文件；
	
	-Z或--compress或--uncompress：通过compress指令处理备份文件；
	
	-f<备份文件>或--file=<备份文件>：指定备份文件；
	
	-v或--verbose：显示指令执行过程；
	
	-r：添加文件到已经压缩的文件；
	
	-u：添加改变了和现有的文件到已经存在的压缩文件；
	
	-j：支持bzip2解压文件；
	
	-v：显示操作过程；
	
	-l：文件系统边界设置；
	
	-k：保留原有文件不覆盖；
	
	-m：保留文件不被覆盖；
	
	-w：确认压缩文件的正确性；
	
	-p或--same-permissions：用原来的文件权限还原文件；
	
	-P或--absolute-names：文件名使用绝对名称，不移除文件名称前的“/”号；
	
	-N <日期格式> 或 --newer=<日期时间>：只将较指定日期更新的文件保存到备份文件里；
	
	--exclude=<范本样式>：排除符合范本样式的文件。

### tar 企业案例演示

	tar cvf jfedu.tar.gz jfedu ：打包 jfedu 文件或者目录，打包后名称为 jfedu.tar.gz
	tar tf jfedu.tar.gz ：查看 jfedu.tar.gz 包中内容
	tar rf jfedu.tar.gz jf du.txt ：将 jfedu.txt 文件追加到 jfedu.tar.gz 
	tar xvf jfedu.tar.gz ：解压 jfedu.tar.gz  程序包
	tar czvf jfedu.tar.gz jfedu ：使用 gzip 格式打包并压缩 jfedu 目录
	tar cjvf jfedu.tar.bz2 jfedu ：使用 bzip2 格式打包并压缩 jfedu 目录
	tar czf jfedu.tar.gz * －X list.txt ：使用 gzip 格式打包并压当前目录所有文件，排除list.txt 中记录的文件
	tar czf jfedu.tar.gz * －-e xclude=zabbix 3.2.4.tar.gz --exclude=nginx-1.12.0.tar.gz ：使用 gzip 格式打包并压缩所有文件和目录，排除 zabbix-3.2.4.tar.gz nginx-1.12.0.tar.gz 软件包
	
	
### tar `增量+差异备份、还原文件`  

**完整备份**:

建立测试路径与档案

	mkdir test
	touch test/{a,b,c}
	
在test下生成三个文件

执行完整备份

	tar -g snapshot -zcf backup_full.tar.gz test

查看 tarball 内容

	tar ztf backup_full.tar.gz
	test/
	test/a
	test/b
	test/c


**差异+增量备份**：

新增一个档案, 并修改一个档案内容

	touch test/e
	echo 123 > test/a

执行第二次的增量备份 (注意 tarball 档名)

	tar -g snapshot -zcf backup_incremental_2.tar.gz test

查看 tarball 内容

	tar ztf backup_incremental_2.tar.gz
	test/
	test/a
	test/e
	
**还原备份资料**:

清空测试资料

	rm -rf test

开始进行资料还原
	
	tar zxf backup_full.tar.gz
	tar zxf backup_incremental_1.tar.gz
	tar zxf backup_incremental_2.tar.gz

查看测试资料

	ls test
	a b c d e

