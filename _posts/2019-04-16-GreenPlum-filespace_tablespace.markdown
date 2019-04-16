---
layout:     post
title:      "Greenplum 表空间和filespace的用法"
subtitle:   "PostgreSQL的表空间概念详解"
date:       2019-04-16
author:     "kikyoar"
header-img: "img/post-bg-linux-version.jpg"
tags:
    - Database
---   

## Greenplum 表空间和filespace的用法

[摘自云栖社区](https://yq.aliyun.com/articles/190)

Greenplum支持表空间，创建表空间时，需要指定filespace  

	postgres=# \h  create tablespace; 
	Command: CREATE TABLESPACE 
	Description: define a new tablespace Syntax: 
	CREATE TABLESPACE tablespace_name [OWNER username] FILESPACE filespace_name 
	
那么什么是filespace呢？ 
GP在初始化完后，有一个默认的filespace:pg\_system 

表空间pg_default和pg_global都放在这个filespace下面  
也就是说一个filespace可以被多个表空间使用    

	postgres=# select oid,* from pg_filespace;
	 oid  |  fsname   | fsowner 
	------+-----------+---------
	 3052 | pg_system |      10
	(1 row)
	
	postgres=# select * from pg_tablespace;
	  spcname   | spcowner | spclocation | spcacl | spcprilocations | spcmirlocations | spcfsoid 
	------------+----------+-------------+--------+-----------------+-----------------+----------
	 pg_default |       10 |             |        |                 |                 |     3052
	 pg_global  |       10 |             |        |                 |                 |     3052
	(2 rows) 


还有TEMPORARY\_FILES和TRANSACTION\_FILES对应的filespace如下：

	$gpfilespace --showtempfilespace
	
	20151218:16:02:07:063949 gpfilespace:127.0.0.1:digoal-[INFO]:-
	A tablespace requires a file system location to store its database
	files. A filespace is a collection of file system locations for all components
	in a Greenplum system (primary segment, mirror segment and master instances).
	Once a filespace is created, it can be used by one or more tablespaces.
	
	20151218:16:02:07:063949 gpfilespace:127.0.0.1:digoal-[INFO]:-Getting filespace information for TEMPORARY_FILES
	20151218:16:02:08:063949 gpfilespace:127.0.0.1:digoal-[INFO]:-Checking for filespace consistency
	20151218:16:02:08:063949 gpfilespace:127.0.0.1:digoal-[INFO]:-Obtaining current filespace entries used by TEMPORARY_FILES
	20151218:16:02:09:063949 gpfilespace:127.0.0.1:digoal-[INFO]:-TEMPORARY_FILES OIDs are consistent for pg_system filespace
	20151218:16:02:11:063949 gpfilespace:127.0.0.1:digoal-[INFO]:-TEMPORARY_FILES entries are consistent for pg_system filespace
	20151218:16:02:11:063949 gpfilespace:127.0.0.1:digoal-[INFO]:-Obtaining current filespace entries used by TEMPORARY_FILES
	20151218:16:02:11:063949 gpfilespace:127.0.0.1:digoal-[INFO]:-Current Filespace for TEMPORARY_FILES is pg_system
	20151218:16:02:11:063949 gpfilespace:127.0.0.1:digoal-[INFO]:-1    /disk1/digoal/gpdata/gpseg-1
	20151218:16:02:11:063949 gpfilespace:127.0.0.1:digoal-[INFO]:-25    /disk1/digoal/gpdata_mirror/gpseg0
	20151218:16:02:11:063949 gpfilespace:127.0.0.1:digoal-[INFO]:-2    /disk1/digoal/gpdata/gpseg0
	......  
	
	$gpfilespace --showtransfilespace
	
	20151218:16:09:41:071104 gpfilespace:127.0.0.1:digoal-[INFO]:-
	A tablespace requires a file system location to store its database
	files. A filespace is a collection of file system locations for all components
	in a Greenplum system (primary segment, mirror segment and master instances).
	Once a filespace is created, it can be used by one or more tablespaces.
	
	20151218:16:09:41:071104 gpfilespace:127.0.0.1:digoal-[INFO]:-Getting filespace information for TRANSACTION_FILES
	20151218:16:09:41:071104 gpfilespace:127.0.0.1:digoal-[INFO]:-Checking for filespace consistency
	20151218:16:09:41:071104 gpfilespace:127.0.0.1:digoal-[INFO]:-Obtaining current filespace entries used by TRANSACTION_FILES
	20151218:16:09:42:071104 gpfilespace:127.0.0.1:digoal-[INFO]:-TRANSACTION_FILES OIDs are consistent for pg_system filespace
	20151218:16:09:44:071104 gpfilespace:127.0.0.1:digoal-[INFO]:-TRANSACTION_FILES entries are consistent for pg_system filespace
	20151218:16:09:44:071104 gpfilespace:127.0.0.1:digoal-[INFO]:-Obtaining current filespace entries used by TRANSACTION_FILES
	20151218:16:09:44:071104 gpfilespace:127.0.0.1:digoal-[INFO]:-Current Filespace for TRANSACTION_FILES is pg_system
	20151218:16:09:44:071104 gpfilespace:127.0.0.1:digoal-[INFO]:-1    /disk1/digoal/gpdata/gpseg-1
	20151218:16:09:44:071104 gpfilespace:127.0.0.1:digoal-[INFO]:-25    /disk1/digoal/gpdata_mirror/gpseg0
	20151218:16:09:44:071104 gpfilespace:127.0.0.1:digoal-[INFO]:-2    /disk1/digoal/gpdata/gpseg0
	20151218:16:09:44:071104 gpfilespace:127.0.0.1:digoal-[INFO]:-26    /disk1/digoal/gpdata_mirror/gpseg1
	......

如果我们的greenplum集群中，有SSD硬盘，又有SATA硬盘。怎样更好的利用这些空间呢？

### **方法1**  

	- 用flashcache或bcache，通过device mapper技术，将SSD和SATA绑定,做成块设备,再通过 逻辑卷管理 或者 软RAID 或者 brtfs or zfs管理起来，做成大的文件系统  
	- 还有一种方法是用RHEL 7提供的LVM，可以将SSD作为二级缓存，这种方法对GP来说，是混合动力，可以创建一个或多个文件系统（都具备混合动力).所以建议只需要一个pg_system filespace就够了(除非容量到了文件系统管理的极限，那样的话可以分成多个文件系统)。用多个文件系统的情况下，就需要对每个文件系统，创建对应的目录，以及filespace

### **方法2**  

	SSD和SATA分开，各自创建各自的文件系统,对每个文件系统，创建对应的目录，以及filespace  
	
创建filespace非常简单:
如下：

- 创建目录，需要在所有的角色对应的主机中创建。给予gp操作系统管理用户对应的权限  

		master
		$ mkdir /ssd1/gpdata/master_p
		$ chown gpadmin:gpadmin /ssd1/gpdata/master_p
		
		master standby
		$ mkdir /ssd1/gpdata/master_s
		$ chown gpadmin:gpadmin /ssd1/gpdata/master_s
		
		segment
		$ mkdir /ssd1/gpdata_p
		$ chown gpadmin:gpadmin /ssd1/gpdata_p
		
		segment mirror
		$ mkdir /ssd1/gpdata_s
		$ chown gpadmin:gpadmin /ssd1/gpdata_s  
			
			
- 查看系统配置  

		postgres=# select dbid,content,role,preferred_role,hostname,port from gp_segment_configuration order by role,dbid;
		 dbid | content | role | preferred_role |     hostname      | port  
		------+---------+------+----------------+-------------------+-------
		    2 |       0 | m    | p              | digoal.sqa.zmf | 40000
		    3 |       1 | m    | p              | digoal.sqa.zmf | 40001
		    4 |       2 | m    | p              | digoal.sqa.zmf | 40002
		    5 |       3 | m    | p              | digoal.sqa.zmf | 40003
		    6 |       4 | m    | p              | digoal.sqa.zmf | 40004
		    7 |       5 | m    | p              | digoal.sqa.zmf | 40005
		    8 |       6 | m    | p              | digoal.sqa.zmf | 40006
		    9 |       7 | m    | p              | digoal.sqa.zmf | 40007
		   10 |       8 | m    | p              | digoal.sqa.zmf | 40008
		   11 |       9 | m    | p              | digoal.sqa.zmf | 40009
		   12 |      10 | m    | p              | digoal.sqa.zmf | 40010
		   13 |      11 | m    | p              | digoal.sqa.zmf | 40011
		   14 |      12 | m    | p              | digoal.sqa.zmf | 40012
		   15 |      13 | m    | p              | digoal.sqa.zmf | 40013
		   16 |      14 | m    | p              | digoal.sqa.zmf | 40014
		   17 |      15 | m    | p              | digoal.sqa.zmf | 40015
		   18 |      16 | m    | p              | digoal.sqa.zmf | 40016
		   19 |      17 | m    | p              | digoal.sqa.zmf | 40017
		   20 |      18 | m    | p              | digoal.sqa.zmf | 40018
		   21 |      19 | m    | p              | digoal.sqa.zmf | 40019
		   22 |      20 | m    | p              | digoal.sqa.zmf | 40020
		   23 |      21 | m    | p              | digoal.sqa.zmf | 40021
		   24 |      22 | m    | p              | digoal.sqa.zmf | 50011
		    1 |      -1 | p    | p              | digoal.sqa.zmf |  1921
		   25 |       0 | p    | m              | digoal.sqa.zmf | 41000
		   26 |       1 | p    | m              | digoal.sqa.zmf | 41001
		   27 |       2 | p    | m              | digoal.sqa.zmf | 41002
		   28 |       3 | p    | m              | digoal.sqa.zmf | 41003
		   29 |       4 | p    | m              | digoal.sqa.zmf | 41004
		   30 |       5 | p    | m              | digoal.sqa.zmf | 41005
		   31 |       6 | p    | m              | digoal.sqa.zmf | 41006
		   32 |       7 | p    | m              | digoal.sqa.zmf | 41007
		   33 |       8 | p    | m              | digoal.sqa.zmf | 41008
		   34 |       9 | p    | m              | digoal.sqa.zmf | 41009
		   35 |      10 | p    | m              | digoal.sqa.zmf | 41010
		   36 |      11 | p    | m              | digoal.sqa.zmf | 41011
		   37 |      12 | p    | m              | digoal.sqa.zmf | 41012
		   38 |      13 | p    | m              | digoal.sqa.zmf | 41013
		   39 |      14 | p    | m              | digoal.sqa.zmf | 41014
		   40 |      15 | p    | m              | digoal.sqa.zmf | 41015
		   41 |      16 | p    | m              | digoal.sqa.zmf | 41016
		   42 |      17 | p    | m              | digoal.sqa.zmf | 41017
		   43 |      18 | p    | m              | digoal.sqa.zmf | 41018
		   44 |      19 | p    | m              | digoal.sqa.zmf | 41019
		   45 |      20 | p    | m              | digoal.sqa.zmf | 41020
		   46 |      21 | p    | m              | digoal.sqa.zmf | 41021
		   47 |      22 | p    | m              | digoal.sqa.zmf | 41022
		(47 rows)

- 创建配置文件，格式如下，假如我要创建一个名为ssd1的filespace  

		字段包含(hostname, dbid, DIR/$prefix + $content)
		$ vi conf
		filespace:ssd1
		digoal.sqa.zmf:1:/ssd1/gpdata/master_p/gp-1
		digoal.sqa.zmf:2:/ssd1/gpdata_p/gp0
		digoal.sqa.zmf:3:/ssd1/gpdata_p/gp1
		......
		digoal.sqa.zmf:25:/ssd1/gpdata_s/gp0
		digoal.sqa.zmf:26:/ssd1/gpdata_s/gp1
		......

		还有一种方法是使用gpfilespace -o conf来生成配置文件。(在提示时输入目录名DIR的部分即可)

- 创建filespace  

		gpfilespace -c conf -h 127.0.0.1 -p 1921 -U digoal -W digoal
			
		20151218:17:16:39:108364 gpfilespace:127.0.0.1:digoal-[INFO]:-Connecting to database
		20151218:17:16:39:108364 gpfilespace:127.0.0.1:digoal-[INFO]:-Filespace "ssd1" successfully created
		......

    然后gpfilespace会自动在数据库执行以下DDL SQL。创建对应的filespace

	**所以我们也可以自己在数据库中执行SQL来创建filespace**  

		CREATE FILESPACE ssd1 
		(
		  1: '/disk1/digoal/new_p/gp-1',
		  2: '/disk1/digoal/new_p/gp0',
		  3: '/disk1/digoal/new_p/gp1',
		  4: '/disk1/digoal/new_p/gp2',
		  5: '/disk1/digoal/new_p/gp3',
		  6: '/disk1/digoal/new_p/gp4',
		  7: '/disk1/digoal/new_p/gp5',
		  8: '/disk1/digoal/new_p/gp6',
		  9: '/disk1/digoal/new_p/gp7',
		  10: '/disk1/digoal/new_p/gp8',
		  11: '/disk1/digoal/new_p/gp9',
		  12: '/disk1/digoal/new_p/gp10',
		  13: '/disk1/digoal/new_p/gp11',
		  14: '/disk1/digoal/new_p/gp12',
		  15: '/disk1/digoal/new_p/gp13',
		  16: '/disk1/digoal/new_p/gp14',
		  17: '/disk1/digoal/new_p/gp15',
		  18: '/disk1/digoal/new_p/gp16',
		  19: '/disk1/digoal/new_p/gp17',
		  20: '/disk1/digoal/new_p/gp18',
		  21: '/disk1/digoal/new_p/gp19',
		  22: '/disk1/digoal/new_p/gp20',
		  23: '/disk1/digoal/new_p/gp21',
		  24: '/disk1/digoal/new_p/gp22',
		  25: '/disk1/digoal/new_s/gp0',
		  26: '/disk1/digoal/new_s/gp1',
		  27: '/disk1/digoal/new_s/gp2',
		  28: '/disk1/digoal/new_s/gp3',
		  29: '/disk1/digoal/new_s/gp4',
		  30: '/disk1/digoal/new_s/gp5',
		  31: '/disk1/digoal/new_s/gp6',
		  32: '/disk1/digoal/new_s/gp7',
		  33: '/disk1/digoal/new_s/gp8',
		  34: '/disk1/digoal/new_s/gp9',
		  35: '/disk1/digoal/new_s/gp10',
		  36: '/disk1/digoal/new_s/gp11',
		  37: '/disk1/digoal/new_s/gp12',
		  38: '/disk1/digoal/new_s/gp13',
		  39: '/disk1/digoal/new_s/gp14',
		  40: '/disk1/digoal/new_s/gp15',
		  41: '/disk1/digoal/new_s/gp16',
		  42: '/disk1/digoal/new_s/gp17',
		  43: '/disk1/digoal/new_s/gp18',
		  44: '/disk1/digoal/new_s/gp19',
		  45: '/disk1/digoal/new_s/gp20',
		  46: '/disk1/digoal/new_s/gp21',
		  47: '/disk1/digoal/new_s/gp22'
		);

   现在你可以使用这个filespace了，例如：

- 将temp , trans移动到这个新的filespace.

		$gpfilespace --movetempfilespace ssd1
		
		20151218:17:17:29:008363 gpfilespace:127.0.0.1:digoal-[INFO]:-
		A tablespace requires a file system location to store its database
		files. A filespace is a collection of file system locations for all components
		in a Greenplum system (primary segment, mirror segment and master instances).
		Once a filespace is created, it can be used by one or more tablespaces.
		
		20151218:17:17:29:008363 gpfilespace:127.0.0.1:digoal-[INFO]:-Database was started in NORMAL mode
		20151218:17:17:29:008363 gpfilespace:127.0.0.1:digoal-[INFO]:-Stopping Greenplum Database
		20151218:17:17:57:008363 gpfilespace:127.0.0.1:digoal-[INFO]:-Starting Greenplum Database in master only mode
		20151218:17:18:02:008363 gpfilespace:127.0.0.1:digoal-[INFO]:-Checking if filespace ssd1 exists
		20151218:17:18:02:008363 gpfilespace:127.0.0.1:digoal-[INFO]:-Checking if filespace is same as current filespace
		20151218:17:18:02:008363 gpfilespace:127.0.0.1:digoal-[INFO]:-Stopping Greenplum Database in master only mode
		20151218:17:18:04:008363 gpfilespace:127.0.0.1:digoal-[INFO]:-Checking for connectivity
		20151218:17:18:04:008363 gpfilespace:127.0.0.1:digoal-[INFO]:-Obtaining current filespace information
		20151218:17:18:04:008363 gpfilespace:127.0.0.1:digoal-[INFO]:-Obtaining current filespace entries used by TEMPORARY_FILES
		20151218:17:18:04:008363 gpfilespace:127.0.0.1:digoal-[INFO]:-Obtaining segment information ...
		20151218:17:18:04:008363 gpfilespace:127.0.0.1:digoal-[INFO]:-Creating RemoteOperations list
		20151218:17:18:04:008363 gpfilespace:127.0.0.1:digoal-[INFO]:-Moving TEMPORARY_FILES filespace from pg_system to ssd1 ...
		20151218:17:18:06:008363 gpfilespace:127.0.0.1:digoal-[INFO]:-Starting Greenplum Database


		$gpfilespace --movetransfilespace ssd1
		...
		20151218:17:19:17:055389 gpfilespace:127.0.0.1:digoal-[INFO]:-Moving TRANSACTION_FILES filespace from pg_system to ssd1 ...
		20151218:17:21:16:055389 gpfilespace:127.0.0.1:digoal-[INFO]:-Starting Greenplum Database  
		

- 创建表空间，使用这个filespace  

		postgres=# create tablespace tbs_ssd1 filespace ssd1;
		CREATE TABLESPACE
		postgres=# create table tt(id int) tablespace tbs_ssd1;
		NOTICE:  Table doesn't have 'DISTRIBUTED BY' clause -- Using column named 'id' as the Greenplum Database data distribution key for this table.
		HINT:  The 'DISTRIBUTED BY' clause determines the distribution of data. Make sure column(s) chosen are the optimal data distribution key to minimize skew.
		CREATE TABLE
		postgres=# select * from pg_tablespace ;
		  spcname   | spcowner | spclocation | spcacl | spcprilocations | spcmirlocations | spcfsoid 
		------------+----------+-------------+--------+-----------------+-----------------+----------
		 pg_default |       10 |             |        |                 |                 |     3052
		 pg_global  |       10 |             |        |                 |                 |     3052
		 tbs_ssd1   |       10 |             |        |                 |                 |    69681
		(3 rows)
		
		postgres=# select * from pg_filespace;
		  fsname   | fsowner 
		-----------+---------
		 pg_system |      10
		 ssd1      |      10
		(2 rows)

greenplum为什么会引入filespace的概念？  

**因为主机目录结构可能不一样，所以原有的目录结构式的方法来创建表空间，可能不够灵活**  

如何查看每个节点的filespace和location的关系？ 

	digoal=# select a.dbid,a.content,a.role,a.port,a.hostname,b.fsname,c.fselocation from gp_segment_configuration a,pg_filespace b,pg_filespace_entry c where a.dbid=c.fsedbid and b.oid=c.fsefsoid order by content;
	 dbid | content | role | port  |     hostname      |  fsname   |             fselocation              
	------+---------+------+-------+-------------------+-----------+--------------------------------------
	    1 |      -1 | p    |  1921 | digoal193096.zmf | pg_system | /data01/gpdata/master_pgdata/gpseg-1
	    2 |       0 | p    | 40000 | digoal193096.zmf | pg_system | /data01/gpdata/gpseg0
	    3 |       1 | p    | 40001 | digoal193096.zmf | pg_system | /data01/gpdata/gpseg1
	    4 |       2 | p    | 40002 | digoal193096.zmf | pg_system | /data01/gpdata/gpseg2
	    5 |       3 | p    | 40000 | digoal199092.zmf | pg_system | /data01/gpdata/gpseg3
	    6 |       4 | p    | 40001 | digoal199092.zmf | pg_system | /data01/gpdata/gpseg4
	    7 |       5 | p    | 40002 | digoal199092.zmf | pg_system | /data01/gpdata/gpseg5
	    8 |       6 | p    | 40000 | digoal200164.zmf | pg_system | /data01/gpdata/gpseg6
	    9 |       7 | p    | 40001 | digoal200164.zmf | pg_system | /data01/gpdata/gpseg7
	   10 |       8 | p    | 40002 | digoal200164.zmf | pg_system | /data01/gpdata/gpseg8
	   11 |       9 | p    | 40000 | digoal204016.zmf | pg_system | /data01/gpdata/gpseg9
	   12 |      10 | p    | 40001 | digoal204016.zmf | pg_system | /data01/gpdata/gpseg10
	   13 |      11 | p    | 40002 | digoal204016.zmf | pg_system | /data01/gpdata/gpseg11
	   14 |      12 | p    | 40000 | digoal204063.zmf | pg_system | /data01/gpdata/gpseg12
	   15 |      13 | p    | 40001 | digoal204063.zmf | pg_system | /data01/gpdata/gpseg13
	   16 |      14 | p    | 40002 | digoal204063.zmf | pg_system | /data01/gpdata/gpseg14
	   17 |      15 | p    | 40003 | digoal193096.zmf | pg_system | /data01/gpdata/gpseg15
	   18 |      16 | p    | 40003 | digoal199092.zmf | pg_system | /data01/gpdata/gpseg16
	   19 |      17 | p    | 40003 | digoal200164.zmf | pg_system | /data01/gpdata/gpseg17
	   20 |      18 | p    | 40003 | digoal204016.zmf | pg_system | /data01/gpdata/gpseg18
	   21 |      19 | p    | 40003 | digoal204063.zmf | pg_system | /data01/gpdata/gpseg19
	   22 |      20 | p    | 40000 | digoal209198.zmf | pg_system | /data01/gpdata/gpseg22
	   23 |      21 | p    | 40001 | digoal209198.zmf | pg_system | /data01/gpdata/gpseg23
	   24 |      22 | p    | 40002 | digoal209198.zmf | pg_system | /data01/gpdata/gpseg24
	   25 |      23 | p    | 40003 | digoal209198.zmf | pg_system | /data01/gpdata/gpseg25
	(25 rows)


## PostgreSQL的表空间  

[摘自博客园](https://www.cnblogs.com/lottu/p/9239535.html)  

**表空间的概念**  

PostgreSQL中的表空间允许在文件系统中定义用来存放表示数据库对象的文件的位置。在PostgreSQL中表空间实际上就是给表指定一个存储目录   
	
**表空间的作用**  

通过使用表空间，管理员可以控制一个PostgreSQL安装的磁盘布局。这么做至少有两个用处：    

- 如果初始化集簇所在的分区或者卷用光了空间，而又不能在逻辑上扩展或者做别的什么操作，那么表空间可以被创建在一个不同的分区上，直到系统可以被重新配置  
- 表空间允许管理员根据数据库对象的使用模式来优化性能。例如，一个很频繁使用的索引可以被放在非常快并且非常可靠的磁盘上，如一种非常贵的固态设备。同时，一个很少使用的或者对性能要求不高的存储归档数据的表可以存储在一个便宜但比较慢的磁盘系统上  

`用一句话来讲：能合理利用磁盘性能和空间,制定最优的物理存储方式来管理数据库表和索引`  

**表空间跟数据库关系**  

- 在Oracle数据库中；一个表空间只属于一个数据库使用；而一个数据库可以拥有多个表空间。属于"一对多"的关系
- 在PostgreSQL集群中；一个表空间可以让多个数据库使用；而一个数据库可以使用多个表空间。属于"多对多"的关系  

**系统自带表空间**  

- 表空间pg\_default是用来存储系统目录对象、用户表、用户表index、和临时表、临时表index、内部临时表的默认空间。对应存储目录\$PADATA/base/
- 表空间pg\_global用来存放系统字典表；对应存储目录\$PADATA/global/  

**查看表空间**  

列出现有的表空间

	postgres=# \db
	             List of tablespaces
	    Name    |  Owner   |      Location       
	------------+----------+---------------------
	 pg_default | postgres | 
	 pg_global  | postgres | 
	 tp_lottu   | lottu    | /data/pg_data/lottu
	(3 rows)
	
	postgres=# select oid,* from pg_tablespace;
	  oid  |  spcname   | spcowner | spcacl | spcoptions 
	-------+------------+----------+--------+------------
	  1663 | pg_default |       10 |        | 
	  1664 | pg_global  |       10 |        | 
	 16385 | tp_lottu   |    16384 |        | 
	(3 rows)


**创建表空间**  

Syntax:

`CREATE TABLESPACE tablespace_name [ OWNER { new_owner | CURRENT_USER | SESSION_USER } ] LOCATION 'directory'`

示例如下：

	postgres=# \c lottu postgres
	You are now connected to database "lottu" as user "postgres".
	
	lottu=# CREATE TABLESPACE tsp01 OWNER lottu LOCATION '/data/pg_data/tsp';
	CREATE TABLESPACE


目录"/data/pg_data/tsp"必须是一个已有的空目录，并且属于PostgreSQL操作系统用户

	$ mkdir -p /data/pg_data/tsp
	$ chown -R postgres:postgres /data/pg_data/tsp  
	
**权限分配**  

表空间的创建本身必须作为一个数据库超级用户完成，但在创建完之后之后你可以允许普通数据库用户来使用它。要这样做，给数据库普通用户授予表空间上的CREATE权限。表、索引和整个数据库都可以被分配到特定的表空间  

示例用户"rax":为普通用户  

	lottu=# \c lottu01 rax
	You are now connected to database "lottu01" as user "rax".
	lottu01=> create table test_tsp(id int) tablespace tsp01;
	ERROR:  permission denied for tablespace tsp01
	
	lottu01=> \c lottu01 postgres
	You are now connected to database "lottu01" as user "postgres".
	
	lottu01=# GRANT CREATE ON TABLESPACE tsp01 TO rax;
	GRANT
	
	lottu01=# \c lottu01 rax
	You are now connected to database "lottu01" as user "rax".
	
	lottu01=> create table test_tsp(id int) tablespace tsp01;
	CREATE TABLE  
	

**为数据库指定默认表空间**  

Syntax:

`ALTER DATABASE name SET TABLESPACE new_tablespace`  

以数据库lottu01为例:

	ALTER DATABASE lottu01 SET TABLESPACE tsp01;
	lottu01=> \c lottu01 lottu
	You are now connected to database "lottu01" as user "lottu".

*注意1：执行该操作；不能连着对应数据库操作*

	lottu01=# ALTER DATABASE lottu01 SET TABLESPACE tsp01;
	ERROR:  cannot change the tablespace of the currently open database
	lottu01=# \c postgres postgres
	You are now connected to database "postgres" as user "postgres".
	
*注意2：执行该操作；对应的数据库不能存在表或者索引已经指定默认的表空间*

	postgres=# ALTER DATABASE lottu01 SET TABLESPACE tsp01;
	ERROR:  some relations of database "lottu01" are already in tablespace "tsp01"
	HINT:  You must move them back to the database's default tablespace before using this command.
	
	postgres=# \c lottu01
	You are now connected to database "lottu01" as user "postgres".
	
	lottu01=# drop table test_tsp ;
	DROP TABLE
	
	lottu01=# create table test_tsp(id int);
	CREATE TABLE
	
	lottu01=# \c postgres postgres 
	You are now connected to database "postgres" as user "postgres".

*注意3：执行该操作；必须是没有人连着对应的数据库*

	postgres=# ALTER DATABASE lottu01 SET TABLESPACE tsp01;
	ERROR:  database "lottu01" is being accessed by other users
	DETAIL:  There is 1 other session using the database.
	postgres=# ALTER DATABASE lottu01 SET TABLESPACE tsp01;
	ALTER DATABASE

查看数据库默认表空间

	lottu01=# select d.datname,p.spcname from pg_database d, pg_tablespace p where d.datname='lottu01' and p.oid = d.dattablespace;
	 datname | spcname 
	---------+---------
	 lottu01 | tsp01
	(1 row)  
	
**如何将表从一个表空间移到另一个表空间**  

我们知道表空间pg\_default是用来存储系统目录对象、用户表、用户表index、和临时表、临时表index、内部临时表的默认空间。若没指定默认表空间；表就所属的表空间就是pg\_default。"当然也可以通过参数设置"。而不是数据库默认的表空间。这个时候我们可以将表移到默认的表空间

Syntax:

`ALTER TABLE name SET TABLESPACE new_tablespace`   

将表从一个表空间移到另一个表空间

	lottu01=# create table test_tsp03(id int) tablespace tp_lottu;
	CREATE TABLE
	
	lottu01=# alter table test_tsp03 set tablespace tsp01;
	ALTER TABLE  
	
`注意：该操作时会锁表`  

**临时表空间**  

PostgreSQL的临时表空间，通过参数temp\_tablespaces进行配置，PostgreSQL允许用户配置多个临时表空间。配置多个临时表空间时，使用逗号隔开。如果没有配置temp\_tablespaces 参数，临时表空间对应的是默认的表空间pg\_default。PostgreSQL的临时表空间用来存储临时表或临时表的索引，以及执行SQL时可能产生的临时文件例如排序，聚合，哈希等。为了提高性能，一般建议将临时表空间放在SSD或者IOPS，以及吞吐量较高的分区中       

	$ mkdir -p /data/pg_data/temp_tsp
	$ chown -R postgres:postgres /data/pg_data/temp_tsp
	postgres=# CREATE TABLESPACE temp01 LOCATION '/data/pg_data/temp_tsp';
	CREATE TABLESPACE
	postgres=# show temp_tablespaces ;
	 temp_tablespaces 
	------------------
	 
	(1 row)
	
*设置临时表空间*

会话级生效  

	postgres=# set temp_tablespaces = 'temp01';
	SET

永久生效

- 修改参数文件postgresql.conf  
- 执行pg_ctl reload  

		[postgres@Postgres201 data]$ grep "temp_tablespace" postgresql.conf
		
		temp_tablespaces = 'temp01'     # a list of tablespace names, '' uses

`查看临时表空间`

	postgres=# show temp_tablespaces ;
	 temp_tablespaces 
	------------------
	 temp01
	(1 row)  
	
