---
layout:     post
title:      "HDFS balance策略详解"
subtitle:   "balancer命令与总结"
date:       2019-03-20
author:     "kikyoar"
header-img: "img/post-bg-hadoop-version.jpg"
tags:
    - Hadoop
---   

[摘自简书](https://www.jianshu.com/p/f7c1cd476601)

线上长时间运行的大规模Hadoop集群，各个datanode节点磁盘空间使用率经常会出现分布不均衡的情况，尤其在新增和下架节点、或者人为干预副本数量的时候。节点空间使用率不均匀会导致计算引擎频繁在跨节点拷贝数据(A节点上运行的task所需数据在其它节点上)，**引起不必要的耗时和带宽**。另外，**当部分节点空间使用率很高但未满(90%左右)时，分配在该节点上的task会存在任务失败的风险**。因此，引入balance策略使集群中的节点空间使用率均匀分布必不可少  

### balancer命令详解

	hdfs --config /hadoop-client/conf balancer
	-threshold  10                    \\集群平衡的条件，datanode间磁盘使用率相差阈值，区间选择：0~100
	-policy datanode                  \\默认为datanode，datanode级别的平衡策略
	-exclude  -f  /tmp/ip1.txt        \\默认为空，指定该部分ip不参与balance， -f：指定输入为文件
	-include  -f  /tmp/ip2.txt        \\默认为空，只允许该部分ip参与balance，-f：指定输入为文件
	-idleiterations  5               \\迭代次数，默认为 5

hdfs balance时datanode之间数据迁移的带宽设置(/hadoop-client/conf/hdfs-site.xml, 修改需重启hdfs)：

	<property>
	    <name>dfs.datanode.balance.bandwidthPerSec</name>
	    <value>6250000</value>
	</property>
	
	<备注：6250000 / (1024 * 1024) = 6M/s>  

动态增大带宽（不需重启，需要切换到hdfs用户，不可设置太大，会占用mapreduce任务的带宽）：  

	hdfs dfsadmin -fs hdfs://${active-namenode-hostname}:8020 -setBalancerBandwidth 104857600  
	
balance脚本在满足以下任何一个条件都会自动退出：  

 * The cluster is balanced
 * No block can be moved
 * No block has been moved for specified consecutive iterations (5 by default)
 * An IOException occurs while communicating with the namenode
 * Another balancer is running  


### 结语   

1. 对于一些大型的HDFS集群(随时可能扩容或下架服务器)，balance脚本需要作为后台常驻进程  
2. 根据官方建议，脚本需要部署在相对空闲的服务器上  
3. 停止脚本通过kill进程实现（建议不kill，后台运行完会自动停止，多次执行同时也只会有一个线程存在，其它自动失败）  


针对datanode存储维护，可以针对以下几个方向进行优化：

* 通过参数(threshold)增加迭代次数，以增加datanode允许迁移的数据    
* 通过参数(exclude, include)设计合理的允许进行balance策略的服务器，比如将使用率最低(20%)和最高(20%)的进行balance策略  
* 通过参数(threshold)设计合理的阈值   