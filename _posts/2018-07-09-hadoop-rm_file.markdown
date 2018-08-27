---
layout:     post
title:      "三种恢复 HDFS 上删除文件的方法"
subtitle:   "Hadoop文件系统元数据fsimage和编辑日志edits解析"
date:       2018-07-09
author:     "kikyoar"
header-img: "img/post-bg-hadoop-version.jpg"
tags:
    - Hadoop
---  

## 通过垃圾箱恢复  

HDFS 为我们提供了垃圾箱的功能，也就是当我们执行 `hadoop fs -rmr xxx` 命令之后，文件并不是马上被删除，而是会被移动到执行这个操作用户的 .Trash 目录下，等到一定的时间后才会执行真正的删除操作。看下下面的例子：  
	
	$ sudo -uiteblog hadoop fs -rmr /user/iteblog/test.txt
	Moved: 'hdfs://iteblogcluster/user/iteblog/test.txt' to trash at: hdfs://iteblogcluster/user/iteblog/.Trash/Current
	 
	$ sudo -uiteblog hadoop fs -ls /user/iteblog/.Trash/Current/user/iteblog
	-rw-r--r--   3 iteblog iteblog  103 2017-05-15 17:24 /user/iteblog/.Trash/Current/user/iteblog/test.txt
	 
	$ sudo -uiteblog hadoop fs -mv /user/iteblog/.Trash/Current/user/iteblog/test.txt /user/iteblog/
	 
	$ sudo -uiteblog hadoop fs -ls /user/iteblog/test.txt
	-rw-r--r--   3 iteblog iteblog  103 2017-05-15 17:24 test.txt  
	
从上面的例子中可以看出，我们删了 test.txt 文件之后，文件被移到 `/user/iteblog/.Trash/Current/user/iteblog/test.txt` 路径下，如果这个操作属于误操作，那么我们可以到回收站找回这个文件并直接 mv 回原来的目录即可恢复之前的数据。hadoop里的trash选项默认是关闭的。所以如果要生效，需要提前将trash选项打开，修改conf里的core-site.xml即可  

    <!--Enabling Trash-->
    <property>
        <name>fs.trash.interval</name>
        <value>1440</value>
    </property>
    <property>
        <name>fs.trash.checkpoint.interval</name>
        <value>1440</value>
    </property>  
    
`fs.trash.interval`是在指在这个回收周期之内，文件实际上是被移动到trash的这个目录下面，而不是马上把数据删除掉。等到回收周期真正到了以后，hdfs才会将数据真正删除。默认的单位是分钟，1440分钟=60\*24，刚好是一天。 `fs.trash.checkpoint.interval`则是指垃圾回收的检查间隔，应该是小于或者等于`fs.trash.interval`，**所以为了误删除操作，强烈建议开启 HDFS 回收站功能**   

## 通过快照恢复  

Hadoop 从 2.1.0 版本开始提供了 HDFS 快照（SnapShot）功能。一个快照是一个全部文件系统、或者某个目录在某一时刻的镜像。利用快照可以防止用户错误操作，管理员可以通过以滚动的方式周期性设置一个只读的快照，这样就可以在文件系统上有若干份只读快照。如果用户意外地删除了一个文件，就可以使用包含该文件的最新只读快照来进行恢复。下面我们来实操说明如何利用快照恢复误删除的文件：

**创建目录和文件**  

	$ sudo -ubizdata hadoop fs -mkdir /user/iteblog/important/
	$ echo "important data" | sudo -uiteblog hadoop fs -put - /user/iteblog/important/important-file.txt
	$ sudo -uiteblog hadoop fs -cat /user/iteblog/important/important-file.txt
	important data  
	
上面我们创建了 /user/iteblog/important/ 目录，里面有一个文件 important-file.txt ，假设这个文件是非常重要的  

**创建快照**  

	$ sudo -uiteblog hadoop dfsadmin -allowSnapshot /user/iteblog/important
	$ sudo -uiteblog hadoop fs -createSnapshot /user/iteblog/important important-snapshot

现在我们已经为 important 目录创建了快照，名称为 important-snapshot。

**误删除操作**

因为开启了快照功能，我们无法删除已经创建快照的目录（/user/iteblog/important），但是我们依然可以删除这个目录下的文件；

	$ sudo -uiteblog hadoop fs -rm -r /user/iteblog/important/important-file.txt

现在这个重要的文件被我们误删除了！

**恢复文件**

别急，因为我们开启了快照，所有我们可以从快照中恢复这个文件，步骤如下：

	$ sudo -uiteblog hadoop fs -ls /user/iteblog/important/.snapshot/
	$ sudo -uiteblog hadoop fs -cp /user/iteblog/important/.snapshot/important-snapshot/important-file.txt /user/iteblog/important/
	$ sudo -uiteblog hadoop fs -cat /user/iteblog/important/important-file.txt
	important data
	
通过上面几步，我们已经恢复了误删除的重要文件  

## 通过编辑日志恢复  

如果你的 Hadoop 集群没有开启回收站功能，也没有对重要的数据创建快照，这时候如果有人将一份非常重要的数据误删除了，那我们如何恢复这些数据？答案是通过修改编辑日志，但是通过这种方法不一定能恢复已经被删除的文件，或者只能恢复一部分被删除的文件，也可能恢复全部误删除的数据，这个和你的集群繁忙状态有很大的关系。而且通过这种方式恢复误删除的文件代价很高，风险很大，需要谨慎使用。下面介绍通过这种恢复删除数据的步骤  

**删除文件**

	sudo -uiteblog hadoop fs -rmr -skipTrash /user/iteblog/important-file.txt

由于上面删除操作使用了 -skipTrash 参数，这意味着这个文件会被直接删除，并不会先放到回收站

**恢复数据**

NameNode 在收到删除命令时，会先将这个命令写到编辑日志中，然后会告诉 DataNode 执行真正的文件删除操作。所以我们需要做的是立刻停止 NameNode 和 DataNode 节点，阻止删除命令的执行。然后找到执行 rmr 操作发生时间对应的编辑日志，假设是 edits_inprogress_0000000000000001512，这个文件是二进制的形式，我们需要通过 HDFS 自带的命令将这个文件转换成可读的形式，如下：

	$ hdfs oev -i edits_inprogress_0000000000000001512 -o edits_inprogress_0000000000000001512.xml

上面执行的结果是二进制的编辑日志被转换成我们人类可读的xml格式的文件，我们找到执行删除 important-file.txt 文件的命令记录：

	<RECORD> 
	  <OPCODE>OP_DELETE</OPCODE>
	  <DATA>
	    <TXID>1624</TXID>
	    <LENGTH>0</LENGTH>
	    <PATH>/user/iteblog/important-file.txt</PATH>
	    <TIMESTAMP>1515724198362</TIMESTAMP>
	    <RPC_CLIENTID>34809cac-a89f-4113-98b5-10c54d7aac1a</RPC_CLIENTID>
	    <RPC_CALLID>1</RPC_CALLID>
	  </DATA> 
	</RECORD>
	
OP_DELETE 这个标记就是删除操作，我们将这个标记修改成比较安全的操作（比如OP_SET_PERMISSIONS），如果这个命令是在最后，可以直接删除，然后保存。再将修改后的编辑日志转换成计算机能够识别的格式：

	$ hdfs oev -i edits_inprogress_0000000000000001512.xml -o edits_inprogress_0000000000000001512 -p binary

最后启动 NameNode 和 DataNode 节点  

## Hadoop文件系统元数据fsimage和编辑日志edits  

NameNode的$dfs.namenode.name.dir/current/文件夹的几个文件：

	current/
	|-- VERSION
	|-- edits_*
	|-- fsimage_0000000000008547077
	|-- fsimage_0000000000008547077.md5
	`-- seen_txid

　　其中存在大量的以edits开头的文件和少量的以fsimage开头的文件。那么这两种文件到底是什么，有什么用？下面对这两中类型的文件进行详解。在进入下面的主题之前先来搞清楚edits和fsimage文件的概念：     

- fsimage文件其实是Hadoop文件系统元数据的一个永久性的检查点，其中包含Hadoop文件系统中的所有目录和文件idnode的序列化信息  

- edits文件存放的是Hadoop文件系统的所有更新操作的路径，文件系统客户端执行的所以写操作首先会被记录到edits文件中  

  
　　fsimage和edits文件都是经过序列化的，在NameNode启动的时候，它会将fsimage文件中的内容加载到内存中，之后再执行edits文件中的各项操作，使得内存中的元数据和实际的同步，存在内存中的元数据支持客户端的读操作

　　NameNode起来之后，HDFS中的更新操作会重新写到edits文件中，因为fsimage文件一般都很大（GB级别的很常见），如果所有的更新操作都往fsimage文件中添加，这样会导致系统运行的十分缓慢，但是如果往edits文件里面写就不会这样，每次执行写操作之后，且在向客户端发送成功代码之前，edits文件都需要同步更新。如果一个文件比较大，使得写操作需要向多台机器进行操作，只有当所有的写操作都执行完成之后，写操作才会返回成功，这样的好处是任何的操作都不会因为机器的故障而导致元数据的不同步。

　　fsimage包含Hadoop文件系统中的所有目录和文件idnode的序列化信息；对于文件来说，包含的信息有修改时间、访问时间、块大小和组成一个文件块信息等；而对于目录来说，包含的信息主要有修改时间、访问控制权限等信息。fsimage并不包含DataNode的信息，而是包含DataNode上块的映射信息，并存放到内存中，当一个新的DataNode加入到集群中，DataNode都会向NameNode提供块的信息，而NameNode会定期的“索取”块的信息，以使得NameNode拥有最新的块映射。因为fsimage包含Hadoop文件系统中的所有目录和文件idnode的序列化信息，所以如果fsimage丢失或者损坏了，那么即使DataNode上有块的数据，但是我们没有文件到块的映射关系，我们也无法用DataNode上的数据！所以定期及时的备份fsimage和edits文件非常重要！

　　文件系统客户端执行的所以写操作首先会被记录到edits文件中，那么久而久之，edits会非常的大，而NameNode在重启的时候需要执行edits文件中的各项操作，那么这样会导致NameNode启动的时候非常长！这样就得了解fsimage和edits合并实现机制（Hadoop1.x和Hadoop2.x是不同的）


## Hadoop 1.x中fsimage和edits合并实现  
   
　　在NameNode运行期间，HDFS的所有更新操作都是直接写到edits中，久而久之edits文件将会变得很大；虽然这对NameNode运行时候是没有什么影响的，但是我们知道当NameNode重启的时候，NameNode先将fsimage里面的所有内容映像到内存中，然后再一条一条地执行edits中的记录，当edits文件非常大的时候，会导致NameNode启动操作非常地慢，而在这段时间内HDFS系统处于安全模式，这显然不是用户要求的。能不能在NameNode运行的时候使得edits文件变小一些呢？其实是可以的，本文主要是针对Hadoop 1.x版本，说明其是怎么将edits和fsimage文件合并的，Hadoop 2.x版本edits和fsimage文件合并是不同的  
   
   
　　用过Hadoop的用户应该都知道在Hadoop里面有个SecondaryNamenode进程，从名字看来大家很容易将它当作NameNode的热备进程。其实真实的情况不是这样的。SecondaryNamenode是HDFS架构中的一个组成部分，它是用来保存namenode中对HDFS metadata的信息的备份，并减少namenode重启的时间而设定的！一般都是将SecondaryNamenode单独运行在一台机器上，那么SecondaryNamenode是如何减少namenode重启的时间的呢？来看看SecondaryNamenode的工作情况：  

- SecondaryNamenode会定期的和NameNode通信，请求其停止使用edits文件，暂时将新的写操作写到一个新的文件edit.new上来，这个操作是瞬间完成，上层写日志的函数完全感觉不到差别

- SecondaryNamenode通过HTTP GET方式从NameNode上获取到fsimage和edits文件，并下载到本地的相应目录下

- SecondaryNamenode将下载下来的fsimage载入到内存，然后一条一条地执行edits文件中的各项更新操作，使得内存中的fsimage保存最新；这个过程就是edits和fsimage文件合并；

- SecondaryNamenode执行完（3）操作之后，会通过post方式将新的fsimage文件发送到NameNode节点上
- NameNode将从SecondaryNamenode接收到的新的fsimage替换旧的fsimage文件，同时将edit.new替换edits文件，通过这个过程edits就变小了！整个过程的执行可以通过下面的图说明：

![fsimage_edits](http://kikyoar.com/img/fsimage_edits.jpg)  

　　在（1）步骤中，我们谈到SecondaryNamenode会定期的和NameNode通信，这个是需要配置的,可以通过core-site.xml进行配置，下面是默认的配置：

	<property>
	  <name>fs.checkpoint.period</name>
	  <value>3600</value>
	  <description>The number of seconds between two periodic checkpoints.
	  </description>
	</property>
	
　　其实如果当fs.checkpoint.period配置的时间还没有到期，我们也可以通过判断当前的edits大小来触发一次合并的操作，可以通过下面配置  

	<property>
	  <name>fs.checkpoint.size</name>
	  <value>67108864</value>
	  <description>The size of the current edit log (in bytes) that triggers
	       a periodic checkpoint even if the fs.checkpoint.period hasn't expired.
	  </description>
	</property>


　　当edits文件大小超过以上配置，即使fs.checkpoint.period还没到，也会进行一次合并。顺便说说SecondaryNamenode下载下来的fsimage和edits暂时存放的路径可以通过下面的属性进行配置：

	<property>
	  <name>fs.checkpoint.dir</name>
	  <value>${hadoop.tmp.dir}/dfs/namesecondary</value>
	  <description>Determines where on the local filesystem the DFS secondary
	      name node should store the temporary images to merge.
	      If this is a comma-delimited list of directories then the image is
	      replicated in all of the directories for redundancy.
	  </description>
	</property>
	 
	<property>
	  <name>fs.checkpoint.edits.dir</name>
	  <value>${fs.checkpoint.dir}</value>
	  <description>Determines where on the local filesystem the DFS secondary
	      name node should store the temporary edits to merge.
	      If this is a comma-delimited list of directoires then teh edits is
	      replicated in all of the directoires for redundancy.
	      Default value is same as fs.checkpoint.dir
	  </description>
	</property>
	
　　**从上面的描述我们可以看出，SecondaryNamenode根本就不是Namenode的一个热备，其只是将fsimage和edits合并。其拥有的fsimage不是最新的，因为在他从NameNode下载fsimage和edits文件时候，新的更新操作已经写到edit.new文件中去了。而这些更新在SecondaryNamenode是没有同步到的！当然，如果NameNode中的fsimage真的出问题了，还是可以用SecondaryNamenode中的fsimage替换一下NameNode上的fsimage，虽然已经不是最新的fsimage，但是我们可以将损失减小到最少！**  
　　
## Hadoop 2.x中fsimage和edits合并实现  


　　在Hadoop 2.x中解决了NameNode的单点故障问题；同时SecondaryName已经不用了，而之前的Hadoop 1.x中是通过SecondaryName来合并fsimage和edits以此来减小edits文件的大小，从而减少NameNode重启的时间。而在Hadoop 2.x中已经不用SecondaryName，那它是怎么来实现fsimage和edits合并的呢？首先我们得知道，在Hadoop 2.x中提供了HA机制（解决NameNode单点故障），可以通过配置奇数个JournalNode来实现HA，HA机制通过在同一个集群中运行两个NN（active NN & standby NN）来解决NameNode的单点故障，在任何时间，只有一台机器处于Active状态；另一台机器是处于Standby状态。Active NN负责集群中所有客户端的操作；而Standby NN主要用于备用，它主要维持足够的状态，如果必要，可以提供快速的故障恢复   
　　
    为了让Standby NN的状态和Active NN保持同步，即元数据保持一致，它们都将会和JournalNodes守护进程通信。当Active NN执行任何有关命名空间的修改，它需要持久化到一半以上的JournalNodes上(通过edits log持久化存储)，而Standby NN负责观察edits log的变化，它能够读取从JNs中读取edits信息，并更新其内部的命名空间。一旦Active NN出现故障，Standby NN将会保证从JNs中读出了全部的Edits，然后切换成Active状态。Standby NN读取全部的edits可确保发生故障转移之前，是和Active NN拥有完全同步的命名空间状态  
    
　　那么这种机制是如何实现fsimage和edits的合并？在standby NameNode节点上会一直运行一个叫做`CheckpointerThread`的线程，这个线程调用StandbyCheckpointer类的doWork()函数，而doWork函数会每隔Math.min(checkpointCheckPeriod, checkpointPeriod)秒来坐一次合并操作，相关代码如下：  
　　
	try {
	          Thread.sleep(1000 * checkpointConf.getCheckPeriod());
	        } catch (InterruptedException ie) {
	}
	 
	public long getCheckPeriod() {
	    return Math.min(checkpointCheckPeriod, checkpointPeriod);
	}
	 
	checkpointCheckPeriod = conf.getLong(
	        DFS_NAMENODE_CHECKPOINT_CHECK_PERIOD_KEY,
	        DFS_NAMENODE_CHECKPOINT_CHECK_PERIOD_DEFAULT);
	         
	checkpointPeriod = conf.getLong(DFS_NAMENODE_CHECKPOINT_PERIOD_KEY, 
	                                DFS_NAMENODE_CHECKPOINT_PERIOD_DEFAULT);
	                                
上面的checkpointCheckPeriod和checkpointPeriod变量是通过获取hdfs-site.xml以下两个属性的值得到：   

	<property>
	  <name>dfs.namenode.checkpoint.period</name>
	  <value>3600</value>
	  <description>The number of seconds between two periodic checkpoints.
	  </description>
	</property>
	 
	<property>
	  <name>dfs.namenode.checkpoint.check.period</name>
	  <value>60</value>
	  <description>The SecondaryNameNode and CheckpointNode will poll the NameNode
	  every 'dfs.namenode.checkpoint.check.period' seconds to query the number
	  of uncheckpointed transactions.
	  </description>
	</property>

当达到下面两个条件的情况下，将会执行一次checkpoint：  

	boolean needCheckpoint = false;
	if (uncheckpointed >= checkpointConf.getTxnCount()) {
	     LOG.info("Triggering checkpoint because there have been " + 
	                uncheckpointed + " txns since the last checkpoint, which " +
	                "exceeds the configured threshold " +
	                checkpointConf.getTxnCount());
	     needCheckpoint = true;
	} else if (secsSinceLast >= checkpointConf.getPeriod()) {
	     LOG.info("Triggering checkpoint because it has been " +
	            secsSinceLast + " seconds since the last checkpoint, which " +
	             "exceeds the configured interval " + checkpointConf.getPeriod());
	     needCheckpoint = true;
	}  
	
当上述needCheckpoint被设置成true的时候，StandbyCheckpointer类的doWork()函数将会调用doCheckpoint()函数正式处理checkpoint。当fsimage和edits的合并完成之后，它将会把合并后的fsimage上传到Active NameNode节点上，Active NameNode节点下载完合并后的fsimage，再将旧的fsimage删掉（Active NameNode上的）同时清除旧的edits文件。步骤可以归类如下：  

- 配置好HA后，客户端所有的更新操作将会写到JournalNodes节点的共享目录中，可以通过下面配置

		<property>
		　　<name>dfs.namenode.shared.edits.dir</name>
		　　<value>qjournal://XXXX/mycluster</value>
		</property>
		 
		<property>
		　　<name>dfs.journalnode.edits.dir</name>
		　　<value>/export1/hadoop2x/dfs/journal</value>
		</property>
	
- Active Namenode和Standby NameNode从JournalNodes的edits共享目录中同步edits到自己edits目录中

- Standby NameNode中的StandbyCheckpointer类会定期的检查合并的条件是否成立，如果成立会合并fsimage和edits文件

- Standby NameNode中的StandbyCheckpointer类合并完之后，将合并之后的fsimage上传到Active NameNode相应目录中

- Active NameNode接到最新的fsimage文件之后，将旧的fsimage和edits文件清理掉

- 通过上面的几步，fsimage和edits文件就完成了合并，由于HA机制，会使得Standby NameNode和Active NameNode都拥有最新的fsimage和edits文件（之前Hadoop 1.x的SecondaryNameNode中的fsimage和edits不是最新的）
　　


