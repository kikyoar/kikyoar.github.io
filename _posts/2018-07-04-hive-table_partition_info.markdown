---
layout:     post
title:      "hive 分区表梳理"
subtitle:   "提高hive的查询效率"
date:       2018-07-04
author:     "kikyoar"
header-img: "img/post-bg-hadoop-version.jpg"
tags:
    - Hadoop
---  

**为了对表进行合理的管理以及提高查询效率，Hive可以将表组织成“分区”，分区是表的部分列的集合，可以为频繁使用的数据建立分区，这样查找分区中的数据时就不需要扫描全表，这对于提高查找效率很有帮助**

**分区是一种根据“分区列”（partition column）的值对表进行粗略划分的机制。Hive中的每个分区对应数据库中相应分区列的一个索引，每个分区对应着表下的一个目录，在HDFS上的表现形式与表在HDFS上的表现形式相同，都是以子目录的形式存在**

**一个表可以在多个维度上进行分区，并且分区可以嵌套使用。建分区需要在创建表时通过PARTITIONED BY子句指定，**例如  

	CREATE TABLE logs(
	timestamp BIGINT,
	line STRING
	)
	PARTITIONED BY (date STRING,country STRING);  
	
**在将数据加载到表内之前，需要数据加载人员明确知道所加载的数据属于哪一个分区。使用分区在某些应用场景下能给有效地提高性能，当只需要遍历某一个小范围内的数据或者一定条件下的数据时，它可以有效减少扫描数据的数量，前提是需要将数据导入到分区内**

**注意：PARTITONED BY子句中定义的列是表中正式的列（分区列），但是数据文件内并不包含这些列**  

## 在Hive里，为什么要分区?

   **庞大的数据集可能需要耗费大量的时间去处理。在许多场景下，可以通过分区或切片的方法减少每一次扫描总数据量，这种做法可以显著地改善性能，数据会依照单个或多个列进行分区，通常按照时间、地域或者是商业维度进行分区。比如vido表，分区的依据可以是电影的种类和评级，另外，按照拍摄时间划分可能会得到更一致的结果。为了达到性能表现的一致性，对不同列的划分应该让数据尽可能均匀分布。最好的情况下，分区的划分条件总是能够对应where语句的部分查询条件。Hive的分区使用HDFS的子目录功能实现。每一个子目录包含了分区对应的列名和每一列的值。但是由于HDFS并不支持大量的子目录，这也给分区的使用带来了限制。我们有必要对表中的分区数量进行预估，从而避免因为分区数量过大带来一系列问题。Hive查询通常使用分区的列作为查询条件。这样的做法可以指定MapReduce任务在HDFS中指定的子目录下完成扫描的工作。HDFS的文件目录结构可以像索引一样高效利用**  
   
   
- 分区表的一个分区对应hdfs上的一个目录
- 分区表包括静态分区表和动态分区表，根据分区会不会自动创建来区分
- 多级分区表，即创建的时候指定 PARTITIONED BY (event_month string,loc string)，根据顺序，级联创建 event_month=XXX/loc=XXX目录，其他和一级的分区表是一样的  

**静态分区表**  

创建静态分区表，加载数据：  

	use test1;
	drop table test1.order_created_partition purge;
	CREATE TABLE order_created_partition (
	order_number string,
	event_time string
	)
	PARTITIONED BY (event_month string)
	ROW FORMAT DELIMITED FIELDS TERMINATED BY "\t";
	-- 建表语句，指定分区字段为event_month，这个字段是伪列
	-- 会在数据load到表的这个分区时，在hdfs上创建名为event_month=2017-12的子目录
	
	LOAD DATA LOCAL INPATH "/tmp/order_created.txt" 
	OVERWRITE INTO TABLE order_created_partition
	PARTITION (event_month='2017-12');
	-- 使用 hdfs dfs -cat .../order_created_partition/event_month=2017-12/order_created.txt
	-- 查看数据文件中并没有event_month这一列
	
	select * from test1.order_created_partition;
	-- 分区表全表扫描，不推荐
	select * from test1.order_created_partition
	 where event_month='2017-12';
	-- 使用where子句，过滤分区字段，遍历某个分区
	-- 以上两个SQL可以查到列event_month信息
	-- 而使用hdfs dfs -cat看不到该列，说明分区表的分区列是伪列
	-- 实际上是hdfs中的分区目录的体现  
	
添加分区，load数据：  

	alter table test1.order_created_partition
	  add partition (event_month='2018-01');
	LOAD DATA LOCAL INPATH "/tmp/order_created.txt" 
	OVERWRITE INTO TABLE order_created_partition
	PARTITION (event_month='2018-01');
	-- 添加一个分区，将一模一样的数据文件加载到该分区
	select * from test1.order_created_partition
	 where event_month='2018-01';
	-- 查到了该分区的记录
	
	LOAD DATA LOCAL INPATH "/tmp/order_created.txt"
	INTO TABLE order_created_partition
	PARTITION (event_month='2018-01');
	select * from test1.order_created_partition
	 where event_month='2018-01';
	-- 不使用OVERWRITE参数，会追加数据到分区
	
	LOAD DATA INPATH "/user/hive/warehouse/test1.db/order_created_partition/event_month=2017-12/order_created.txt"
	INTO TABLE order_created_partition
	PARTITION (event_month='2018-01');
	-- 如果从hdfs中加载数据，则原来的路径文件被转移掉  
	
*可以看出，所谓的分区，是人为定义的，跟业务数据实际上是不是属于该分区没关系，比如将相同的数据分别插入两个分区中，再比如插入的数据有2017-12月份的和2018-01月份的数据*  

删除分区：  

	alter table test1.order_created_partition
	  drop partition (event_month='2018-01');
	-- 从hdfs中已经看不到event_month=2018-01的分区子目录了  
	
查询装入数据：

	insert overwrite table order_created_partition 
	 partition(event_month='2017-11')
	select order_number,event_time
	  from order_created_partition 
	 where event_month='2018-02';
	-- 使用查询装入数据到分区中，分区可以是当前不存在的
	-- 因为查询的是分区表，需要注意伪列和被装入的分区表列的对应关系  
	
手工创建hdfs目录和文件，添加分区的情况：  
 
静态分区表如果手工创建对应的hdfs目录上传文件，而不使用分区创建命令和load数据到分区的命令，分区表中无法查到该分区信息，需要刷新，这种添加分区的途径是不合法的：  

	hdfs dfs -mkdir -p /user/hive/warehouse/test1.db/order_created_partition/event_month=2018-02
	hdfs dfs -put /tmp/order_created.txt  /user/hive/warehouse/test1.db/order_created_partition/event_month=2018-02
	hdfs dfs -ls /user/hive/warehouse/test1.db/order_created_partition/event_month=2018-02
	
	hive
	
	select * from test1.order_created_partition
	 where event_month='2018-02';
	-- 此时是查不到该分区的
	MSCK REPAIR TABLE order_created_partition;
	-- 修复表信息之后可以查询
	show partitions order_created_partition;
	-- 查看该表的所有分区  
	
**动态分区表**  

	use test1;
	select * from emp;
	-- 根据从前实验创建的emp表
	-- 将emp表的数据按照部门分组，并将数据加载到其对应的分组中去
	create table emp_partition(
	empno int, 
	ename string, 
	job string, 
	mgr int, 
	hiredate string, 
	sal double, 
	comm double)
	PARTITIONED BY (deptno int)
	row format delimited fields terminated by '\t';
	-- 根据部门编号分区，原表中的部门编号字段就没有必要创建了
	-- 而是由分区表创建的伪列来替代
	set hive.exec.dynamic.partition.mode=nonstrict;
	-- 设置动态分区模式为非严格模式
	insert into table emp_partition partition(deptno)
	select empno,ename,job,mgr,hiredate,sal,comm ,deptno from emp;
	-- 动态分区表的数据插入语句
	-- partition(deptno) 而不是 partition(deptno=XXX)
	-- select 子句从原表查出来的列数和列序要和分区表列数和列序保持一致
	-- select 子句最后一列要为分区表的分区列
	-- 不在需要where子句
	-- 设置动态分区模式为非严格模式
	-- set hive.exec.dynamic.partition.mode=nonstrict;
	
## 关于分区的表操作的注意事项  

- 一般来说，查询分区表时，一定会在where子句中加上分区条件，指明查看哪个分区的数据。否则会报错。因为默认set hive.mapred.mode=strict.即严格模式。如果set hive.mapred.mode=nostrict.可以查询分区表时不带分区声明，这个时候会返回整张表的所有数据

- 往分区表中load数据或者导出数据时，要指定分区

 load data local inpath'/home/robot/111.txt' overwrite into table  FDM_SOR.mytest_deptaddr partition(statis_date='20180228')