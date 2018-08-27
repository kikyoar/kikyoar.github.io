---
layout:     post
title:      "hive自动关闭的处理办法"
subtitle:   "hadoop /tmp文件空间占满解决办法"
date:       2018-07-09
author:     "kikyoar"
header-img: "img/post-bg-hadoop-version.jpg"
tags:
    - Hadoop
---    


**问题原因：**

dbeaver配置均没问题，但是连接hive出现timeout的报错，最初以为是hive client问题，想的是重启hive集群可能会清空tmp下的hive文件，重启后一个节点出现问题  

![hive自动关闭](http://kikyoar.com/img/hive_shutdown_01.png)  

在Ambari界面重新启动后，过一会又自动会down，登录该服务器查阅日志，目录为`/var/log/hive`   

	tail -200 hiveserver2.log  
	
![hive自动关闭](http://kikyoar.com/img/hive_shutdown_02.png)

其中错误原因为`dfs.namenode.fs-limits.max-directory-items`默认为1048576  

## 解决办法1  

遂根据日志删除/tmp/hive/hive下前13天的日志  

	#!/bin/bash
	#source ~/.bashrc
	# HADOOP所在的bin目录
	HADOOP_BIN_PATH=/usr/hdp/2.6.3.0-235/hadoop-hdfs/bin
	#待检测的HDFS目录
	data1_file=/tmp/hive/hive
	#将待检测的目录(可以为多个)加载至数组中
	array_check=($data1_file)
	# 当前时间戳
	today_timestamp=$(date -d "$(date +"%Y-%m-%d %H:%M")" +%s)
	#Func: 删除指定时间之前的过期，这里设置的是13天前
	removeOutDate(){
	        $HADOOP_BIN_PATH/hdfs dfs  -ls $1 > temp.txt
	        cat temp.txt | while read quanxian temp user group size day hour filepath
	        do
	            current_file_time="$day $hour"
	            current_file_timestamp=$(date -d "$current_file_time" +%s)
	            if [ $(($today_timestamp-$current_file_timestamp)) -ge $((12*24*60*60)) ];then
	                echo "$(date +'%Y-%m-%d %H:%M:%S') $filepath"
	                $HADOOP_BIN_PATH/hdfs dfs -rm -r $filepath > /dev/null 2>&1
	            fi
	        done
	}
	#Func: 执行删除
	execute(){
	        echo -e "\n\n"
	        echo "$(date +'%Y-%m-%d %H:%M:%S') start to remove outdate files in hdfs"
	        echo "$(date +'%Y-%m-%d %H:%M:%S') today is: $(date +"%Y-%m-%d %H:%M:%S")"
	        for i in ${array_check[@]}
	        do
	            echo "$(date +'%Y-%m-%d %H:%M:%S') processing filepath: $i"
	            removeOutDate $i
	            echo -e "\n"
	        done
	        echo "$(date +'%Y-%m-%d %H:%M:%S') remove outdate files in hdfs finished"
	        echo -e "\n\n"
	}
	# 开始执行
	execute  
	
出现以下错误：  

	2018-07-09 15:28:18 processing filepath: /tmp/hive/hive
	Exception in thread "main" java.lang.OutOfMemoryError: GC overhead limit exceeded
	 	at java.util.Arrays.copyOfRange(Arrays.java:3664)
	 	at java.lang.String.<init>(String.java:207)
	 	at java.lang.StringBuilder.toString(StringBuilder.java:407)
	 	at org.apache.hadoop.fs.Path.<init>(Path.java:109)
	 	at org.apache.hadoop.fs.Path.<init>(Path.java:93)
	 	at org.apache.hadoop.hdfs.protocol.HdfsFileStatus.getFullPath(HdfsFileStatus.java:230)
	 	at org.apache.hadoop.hdfs.protocol.HdfsFileStatus.makeQualified(HdfsFileStatus.java:268)
	 	at org.apache.hadoop.hdfs.DistributedFileSystem.listStatusInternal(DistributedFileSystem.java:961)
	 	at org.apache.hadoop.hdfs.DistributedFileSystem.access$600(DistributedFileSystem.java:114)
	 	at org.apache.hadoop.hdfs.DistributedFileSystem$22.doCall(DistributedFileSystem.java:985)
	 	at org.apache.hadoop.hdfs.DistributedFileSystem$22.doCall(DistributedFileSystem.java:981)
	 	at org.apache.hadoop.fs.FileSystemLinkResolver.resolve(FileSystemLinkResolver.java:81)
	 	at org.apache.hadoop.hdfs.DistributedFileSystem.listStatus(DistributedFileSystem.java:992)
	 	at org.apache.hadoop.fs.shell.PathData.getDirectoryContents(PathData.java:268)
	 	at org.apache.hadoop.fs.shell.Command.recursePath(Command.java:373)
	 	at org.apache.hadoop.fs.shell.Ls.processPathArgument(Ls.java:220)
	 	at org.apache.hadoop.fs.shell.Command.processArgument(Command.java:271)
	 	at org.apache.hadoop.fs.shell.Command.processArguments(Command.java:255)
	 	at org.apache.hadoop.fs.shell.FsCommand.processRawArguments(FsCommand.java:119)
	 	at org.apache.hadoop.fs.shell.Command.run(Command.java:165)
	 	at org.apache.hadoop.fs.FsShell.run(FsShell.java:297)
	 	at org.apache.hadoop.util.ToolRunner.run(ToolRunner.java:76)
	 	at org.apache.hadoop.util.ToolRunner.run(ToolRunner.java:90)
	 	at org.apache.hadoop.fs.FsShell.main(FsShell.java:356)  
	 	
修改集群外一台安装了hdfs client的参数，`vi /etc/hadoop/2.6.3.0-235/0/hadoop-env.sh`

	export HADOOP_HEAPSIZE=“5376"  
	
重新执行该脚本

![hive自动关闭](http://kikyoar.com/img/hive_shutdown_03.png)  

删除特别慢，请耐心等待，这样可以解决以上问题  



## 解决办法2  

**另一种方法是 ：hdfs-site.xml中添加以下，最大不能超过6400000**  

	<property>
	<name>dfs.namenode.fs-limits.max-directory-items</name>
	<value>4194304</value>
	</property>  
	
以下为Ambari界面操作步骤：  

![hive自动关闭](http://kikyoar.com/img/hive_shutdown_04.png) 

****

![hive自动关闭](http://kikyoar.com/img/hive_shutdown_05.png) 

****

![hive自动关闭](http://kikyoar.com/img/hive_shutdown_06.png) 

添加属性内容为：`dfs.namenode.fs-limits.max-directory-items=4194304`   


保存后，开始按顺序重启以下服务

**Hdfs------yarn------MapReduce2------hive**


以上故障解决完成