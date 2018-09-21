---
layout:     post
title:      "Apache Hadoop 机架感知"
subtitle:   "如何为CDH集群配置机架感知"
date:       2018-09-21
author:     "kikyoar"
header-img: "img/post-bg-hadoop-version.jpg"
tags:
    - Hadoop
---  


## 背景  

   分布式的集群通常包含非常多的机器，由于受到机架槽位和交换机网口的限制，通常大型的分布式集群都会跨好几个机架，由多个机架上的机器共同组成一个分布式集群。机架内的机器之间的网络速度通常都会高于跨机架机器之间的网络速度，并且机架之间机器的网络通信通常受到上层交换机间网络带宽的限制  

   具体到Hadoop集群，由于hadoop的HDFS对数据文件的分布式存放是按照分块block存储，每个block会有多个副本(默认为3)，并且为了数据的安全和高效，所以hadoop默认对3个副本的存放策略为：  

- **第一个block副本放在和client所在的node里（如果client不在集群范围内，则这第一个node是随机选取的）**

- **第二个副本放置在与第一个节点不同的机架中的node中（随机选择）**  

- **第三个副本放置在与第二个副本所在节点同一机架的另一个节点上**

- **如果还有更多的副本就随机放在集群的node里** 

这样的策略可以保证对该block所属文件的访问能够优先在本rack下找到，如果整个rack发生了异常，也可以在另外的rack上找到该block的副本。这样足够的高效，并且同时做到了数据的容错

但是，hadoop对机架的感知并非是自适应的，亦即，hadoop集群分辨某台slave机器是属于哪个rack并非是只能的感知的，而是需要hadoop的管理者人为的告知hadoop哪台机器属于哪个rack，这样在hadoop的namenode启动初始化时，会将这些机器与rack的对应信息保存在内存中，用来作为对接下来所有的HDFS的写块操作分配datanode列表时（比如3个block对应三台datanode）的选择datanode策略，做到hadoop allocate block的策略：**尽量将三个副本分布到不同的rack**

接下来的问题就是：通过什么方式能够告知hadoop namenode哪些slaves机器属于哪个rack？以下是配置步骤  

## 配置  

默认情况下，hadoop的机架感知（Rack Awareness）是没有被启用的。所以，在通常情况下，hadoop集群的HDFS在选机器的时候，是随机选择的，也就是说，**很有可能在写数据时，hadoop将第一块数据block1写到了rack1上，然后随机的选择下将block2写入到了rack2下，此时两个rack之间产生了数据传输的流量，再接下来，在随机的情况下，又将block3重新又写回了rack1，此时，两个rack之间又产生了一次数据流量。在job处理的数据量非常的大，或者往hadoop推送的数据量非常大的时候，这种情况会造成rack之间的网络流量成倍的上升，成为性能的瓶颈，进而影响作业的性能以至于整个集群的服务**

要将hadoop机架感知的功能启用，配置非常简单，在namenode所在机器的hadoop-site.xml配置文件中配置一个选项：

`topology.script.file.name`

`/path/to/RackAware.py`


这个配置选项的value指定为一个可执行程序，通常为一个脚本，该脚本接受一个参数，输出一个值。接受的参数通常为某台datanode机器的ip地址，而输出的值通常为该ip地址对应的datanode所在的rack，例如”/rack1”。Namenode启动时，会判断该配置选项是否为空，如果非空，则表示已经用机架感知的配置，此时namenode会根据配置寻找该脚本，并在接收到每一个datanode的heartbeat时，将该datanode的ip地址作为参数传给该脚本运行，并将得到的输出作为该datanode所属的机架，保存到内存的一个map中  

**网络拓扑机器之间的距离**

这里基于一个网络拓扑案例，介绍在复杂的网络拓扑中hadoop集群每台机器之间的距离

![hadoop机架感知](http://kikyoar.com/img/hadoop_rack.jpg)   

有了机架感知，NameNode就可以画出上图所示的datanode网络拓扑图。D1,R1都是交换机，最底层是datanode。则H1的rackid=/D1/R1/H1，H1的parent是R1，R1的是D1。这些rackid信息可以通过topology.script.file.name配置。有了这些rackid信息就可以计算出任意两台datanode之间的距离

	distance(/D1/R1/H1,/D1/R1/H1)=0 相同的datanode
	
	distance(/D1/R1/H1,/D1/R1/H2)=2 同一rack下的不同datanode
	
	distance(/D1/R1/H1,/D1/R1/H4)=4 同一IDC下的不同datanode
	
	distance(/D1/R1/H1,/D2/R3/H7)=6 不同IDC下的datanode  
	

## CDH机架感知操作步骤  


默认情况下，机架感知是没有启用的,这时任何一台 Datanode 节点，不管物理上是否属于同一个机架，Namenode 都会默认将他们划分到/default-rack下  

配置机架感知需要人为地告诉Namenode哪台Datanode位于哪个机架下，将真实的网络拓朴和机架信息了解清楚后，通过机架感知脚本将机器的IP地址正确的映射到相应的机架上去，使逻辑机架与物理机架保持一致

需要注意的是，在Namenode上，必须使用IP，使用主机名无效，而YARN上，必须使用主机名，使用IP无效，所以，建议IP和主机名都配上。配置机架感知后，如果需要新加入Datanode，则需要将Datanode节点信息添加到机架感知脚本中，否则Datanode日志中会有异常信息，且Datanode启动不成功  


**前置准备**  

编辑RackAware.py脚本，添加python脚本的必要代码，最终脚本内容如下：  

	root@bigdata60:/root>cat genRack.sh
	#!/bin/sh
	
	
	root@bigdata60:/root>cat RackAware.py
	
	#!/usr/bin/python
	
	#-*-coding:UTF-8 -*-
	
	import sys
	
	rack = {"bigdata1":"/k0809/k08",
	
	     "bigdata2":"/k0809/k08",
	
	     "bigdata3":"/k0809/k08",
	
	     "bigdata4":"/k0809/k08",
	
	     "bigdata5":"/k0809/k08",
	
	     "bigdata6":"/k0809/k08",
	
	     "bigdata7":"/k0809/k08",
	
	     "bigdata8":"/k0809/k09",
	
	     "bigdata9":"/k0809/k09",
	
	     "bigdata10":"/k0809/k09",
	
	     "bigdata11":"/k0809/k09",
	
	     "bigdata12":"/k0809/k09",
	
	     "bigdata13":"/k0809/k09",
	
	     "bigdata14":"/k0809/k09",
	
	     "bigdata15":"/k1011/k10",
	
	     "bigdata16":"/k1011/k10",
	
	     "bigdata17":"/k1011/k10",
	
	     "bigdata18":"/k1011/k10",
	
	     "bigdata19":"/k1011/k10",
	
	     "bigdata20":"/k1011/k10",
	
	     "bigdata21":"/k1011/k10",
	
	     "bigdata22":"/k1011/k10",
	
	     "bigdata23":"/k1011/k11",
	
	     "bigdata24":"/k1011/k11",
	
	     "bigdata25":"/k1011/k11",
	
	     "bigdata26":"/k1011/k11",
	
	     "bigdata27":"/k1011/k11",
	
	     "bigdata28":"/k1011/k11",
	
	     "bigdata29":"/k1011/k11",
	
	     "bigdata30":"/k1011/k11",
	
	     "bigdata31":"/k0809/j03",
	
	     "bigdata32":"/k0809/j03",
	
	     "bigdata33":"/k0809/j03",
	
	     "bigdata34":"/k0809/j03",
	
	     "bigdata35":"/k0809/j03",
	
	     "bigdata36":"/k0809/j03",
	
	     "bigdata37":"/k0809/j03",
	
	     "bigdata38":"/k0809/j03",
	
	     "bigdata39":"/k0809/j04",
	
	     "bigdata40":"/k0809/j04",
	
	     "bigdata41":"/k0809/j04",
	
	     "bigdata42":"/k0809/j04",
	
	     "bigdata43":"/k0809/j04",
	
	     "bigdata44":"/k0809/j04",
	
	     "bigdata45":"/k0809/j04",
	
	     "bigdata46":"/k0809/j04",
	
	     "bigdata47":"/k1011/j05",
	
	     "bigdata48":"/k1011/j05",
	
	     "bigdata49":"/k1011/j05",
	
	     "bigdata50":"/k1011/j05",
	
	     "bigdata51":"/k1011/j05",
	
	     "bigdata52":"/k1011/j05",
	
	     "bigdata53":"/k1011/j05",
	
	     "bigdata54":"/k1011/j06",
	
	     "bigdata55":"/k1011/j06",
	
	     "bigdata56":"/k1011/j06",
	
	     "bigdata57":"/k1011/j06",
	
	     "bigdata58":"/k1011/j06",
	
	     "bigdata59":"/k1011/j06",
	
	     "bigdata60":"/k1011/j06",
	
	     "bigdata-job-1":"/bigdata/default",
	
	     "bigdata-job-2":"/bigdata/default",
	
	     "bigdata-job-3":"/bigdata/default",
	
	     "bigdata-mysql-1":"/bigdata/default",
	
	     "bigdata-python-1":"/bigdata/default",
	
	     "bigdata-python-2":"/bigdata/default",
	
	     "cm1":"/bigdata/default",
	
	     "cm2":"/bigdata/default",
	
	     "192.***.***.1":"/k0809/k08",
	
	     "192.***.***.2":"/k0809/k08",
	
	     "192.***.***.3":"/k0809/k08",
	
	     "192.***.***.4":"/k0809/k08",
	
	     "192.***.***.5":"/k0809/k08",
	
	     "192.***.***.6":"/k0809/k08",
	
	     "192.***.***.7":"/k0809/k08",
	
	     "192.***.***.8":"/k0809/k09",
	
	     "192.***.***.9":"/k0809/k09",
	
	     "192.***.***.10":"/k0809/k09",
	
	     "192.***.***.11":"/k0809/k09",
	
	     "192.***.***.12":"/k0809/k09",
	
	     "192.***.***.13":"/k0809/k09",
	
	     "192.***.***.14":"/k0809/k09",
	
	     "192.***.***.15":"/k1011/k10",
	
	     "192.***.***.16":"/k1011/k10",
	
	     "192.***.***.17":"/k1011/k10",
	
	     "192.***.***.18":"/k1011/k10",
	
	     "192.***.***.19":"/k1011/k10",
	
	     "192.***.***.20":"/k1011/k10",
	
	     "192.***.***.21":"/k1011/k10",
	
	     "192.***.***.22":"/k1011/k10",
	
	     "192.***.***.23":"/k1011/k11",
	
	     "192.***.***.24":"/k1011/k11",
	
	     "192.***.***.25":"/k1011/k11",
	
	     "192.***.***.26":"/k1011/k11",
	
	     "192.***.***.27":"/k1011/k11",
	
	     "192.***.***.28":"/k1011/k11",
	
	     "192.***.***.29":"/k1011/k11",
	
	     "192.***.***.30":"/k1011/k11",
	
	     "192.***.***.31":"/k0809/j03",
	
	     "192.***.***.32":"/k0809/j03",
	
	     "192.***.***.33":"/k0809/j03",
	
	     "192.***.***.34":"/k0809/j03",
	
	     "192.***.***.35":"/k0809/j03",
	
	     "192.***.***.36":"/k0809/j03",
	
	     "192.***.***.37":"/k0809/j03",
	
	     "192.***.***.38":"/k0809/j03",
	
	     "192.***.***.39":"/k0809/j04",
	
	     "192.***.***.40":"/k0809/j04",
	
	     "192.***.***.41":"/k0809/j04",
	
	     "192.***.***.42":"/k0809/j04",
	
	     "192.***.***.43":"/k0809/j04",
	
	     "192.***.***.44":"/k0809/j04",
	
	     "192.***.***.45":"/k0809/j04",
	
	     "192.***.***.46":"/k0809/j04",
	
	     "192.***.***.47":"/k1011/j05",
	
	     "192.***.***.48":"/k1011/j05",
	
	     "192.***.***.49":"/k1011/j05",
	
	     "192.***.***.50":"/k1011/j05",
	
	     "192.***.***.51":"/k1011/j05",
	
	     "192.***.***.52":"/k1011/j05",
	
	     "192.***.***.53":"/k1011/j05",
	
	     "192.***.***.54":"/k1011/j06",
	
	     "192.***.***.55":"/k1011/j06",
	
	     "192.***.***.56":"/k1011/j06",
	
	     "192.***.***.57":"/k1011/j06",
	
	     "192.***.***.58":"/k1011/j06",
	
	     "192.***.***.59":"/k1011/j06",
	
	     "192.***.***.60":"/k1011/j06",
	
	     "192.***.***.23":"/bigdata/default",
	
	     "192.***.***.24":"/bigdata/default",
	
	     "192.***.***.25":"/bigdata/default",
	
	     "192.***.***.16":"/bigdata/default",
	
	     "192.***.***.21":"/bigdata/default",
	
	     "192.***.***.22":"/bigdata/default",
	
	     "192.***.***.156":"/bigdata/default",
	
	     "192.***.***.157":"/bigdata/default",
	
	     }
	
	
	
	if __name__=="__main__":
	
	#  获取机架信息，如果不存在，则返回"/bigdata/default"
	
	print rack.get(sys.argv[1],"/bigdata/default")


**操作步骤**   

1、将RackAware.py这个机架感知脚本存放在Namenode本次磁盘上，并为脚本赋予执行权限，如果启用了HA，则两个Namenode都需要在本地存放机架感知脚本且存放路径必须相同，避免Namenode发生故障切换导致机架信息不一致  


2、在CM管理界面，为所有主机分配逻辑机架，与机架感知脚本配置的机架信息保持一致

3、登录CM管理界面，在HDFS配置项搜索“topology”，指定机架感知脚本在Namenode节点上的存放路径，保存更改后重启HDFS  

**总结**  

- 配置机架感知需要编写机架感知脚本并赋予执行权限，NameNode通过机架感知脚本，可以确定每个DataNode所属的机架ID，通过CM管理界面为所有节点分配逻辑机架，通常逻辑机架和物理机架一一对应，如果配置逻辑机架包含两层（交换机和机架），那么建议所有逻辑机架都要配置两层，不能出现主机配置逻辑机架层数不一致的情况，否则CM会有告警提示。假如物理机上做了虚拟化，那么虚拟化这一层也要配置到逻辑机架中

- 如果集群启用了HA，则两个Namenode节点需要在本地磁盘的相同路径下存放机架感知脚本。通过CM管理界面，配置HDFS的“topology”参数，指定机架感知脚本存放路径

- 机架感知脚本不能存放在“/etc/hadoop/conf.cloudera.hdfs/”目录下，否则重新部署HDFS客户端配置时，会将机架感知脚本清除。现已将机架感知脚本存放在“/opt/cloudera/parcels/CDH/”目录下  

- 机架感知脚本也可以参考官方给的实例，使用Shell脚本编写，具体参看：
`https://wiki.apache.org/hadoop/topology_rack_awareness_scripts`

- 配置机架感知后，使用SparkSQL可能会出现如下警告信息：  

		18/07/20 09:31:32 WARN net.ScriptBasedMapping: Exception running  /opt/cloudera/parcels/CDH/RackAware.py 192.***.***.35
		
		java.io.IOException: Cannot run program  "/opt/cloudera/parcels/CDH/RackAware.py" (in directory  "/home/iotcdh"): error=2, 没有那个文件或目录  
	
	如果出现该问题，将机架感知脚本拷贝到部署了Spark角色实例的所有节点上，与Namenode存放机架感知脚本的路径保持一致
	
