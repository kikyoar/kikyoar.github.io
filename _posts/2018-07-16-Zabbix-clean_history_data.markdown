---
layout:     post
title:      "zabbix清理历史数据"
subtitle:   "Mysql查看数据库,表占用磁盘大小"
date:       2018-07-16
author:     "kikyoar"
header-img: "img/post-bg-linux-version.jpg"
tags:
    - zabbix
---  

## 查看数据库,表占用磁盘大小  

查询所有数据库占用磁盘空间大小  

	select 
	TABLE_SCHEMA, 
	concat(truncate(sum(data_length)/1024/1024,2),' MB') as data_size,
	concat(truncate(sum(index_length)/1024/1024,2),'MB') as index_size
	from information_schema.tables
	group by TABLE_SCHEMA
	ORDER BY data_size desc;
	#order by data_length desc;  
	
查询单个库中所有表磁盘占用大小

	select 
	TABLE_NAME, 
	concat(truncate(data_length/1024/1024,2),' MB') as data_size,
	concat(truncate(index_length/1024/1024,2),' MB') as index_size
	from information_schema.tables 
	where TABLE_SCHEMA = 'mysql'
	group by TABLE_NAME
	order by data_length desc;


## zabbix自动清理60天前的数据  

	#!/bin/bash
	User="root"
	Passwd="密码"
	Date=`date -d $(date -d "-60 day" +%Y%m%d) +%s` #取60天之前的时间戳
	$(which mysql) -u${User} -p${Passwd} -e "
	use zabbix;
	DELETE FROM history WHERE clock < $Date;
	optimize table history;
	DELETE FROM history_str WHERE clock < $Date;
	optimize table history_str;
	DELETE FROM history_uint WHERE clock < $Date;
	optimize table history_uint;
	DELETE FROM  trends WHERE clock < $Date;
	optimize table  trends;
	DELETE FROM trends_uint WHERE clock < $Date;
	optimize table trends_uint;
	DELETE FROM events WHERE clock < $Date;
	optimize table events;
	"  

**zabbix属于一个细度化的监控工具，其入库数据随着细度的增加相应的入库数据量也会较大，当数据量到一定时候的时候其反映速度会比较慢，尽管其监控服务在配置时可以指定数据的保存周期， 但是了解下通过直接操作数据库进行数据删除还是有必要的；其中histroy是详细的历史数据，trends是图表趋势数据**

	
## mysql中OPTIMIZE TABLE的作用及使用  

	OPTIMIZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ...  
	
如果已经删除了表的一大部分，或者如果您已经对含有可变长度行的表（含有VARCHAR, BLOB或TEXT列的表）进行了很多更改，则应使用OPTIMIZE TABLE。被删除的记录被保持在链接清单中，后续的INSERT操作会重新使用旧的记录位置。您可以使用OPTIMIZE TABLE来重新利用未使用的空间，并整理数据文件的碎片

在多数的设置中，您根本不需要运行OPTIMIZE TABLE。即使您对可变长度的行进行了大量的更新，您也不需要经常运行，每周一次或每月一次即可，只对特定的表运行

OPTIMIZE TABLE只对MyISAM, BDB和InnoDB表起作用

注意，在OPTIMIZE TABLE运行过程中，MySQL会锁定表  

**一、原始数据**

1、数据量

	mysql> select count(*) as total from ad_visit_history; 
	+---------+ 
	| total | 
	+---------+ 
	| 1187096 | //总共有118万多条数据 
	+---------+ 
	1 row in set (0.04 sec)

2、存放在硬盘中的表文件大小

	[root@ www.linuxidc.com test1]# ls |grep visit |xargs -i du {} 
	382020 ad_visit_history.MYD //数据文件占了380M 
	127116 ad_visit_history.MYI //索引文件占了127M 
	12 ad_visit_history.frm //结构文件占了12K

3、查看一下索引信息

	mysql> show index from ad_visit_history from test1; //查看一下该表的索引信息 
	+------------------+------------+-------------------+--------------+---------------+-----------+-------------+----------+--------+------+------------+---------+ 
	| Table | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | 
	+------------------+------------+-------------------+--------------+---------------+-----------+-------------+----------+--------+------+------------+---------+ 
	| ad_visit_history | 0 | PRIMARY | 1 | id | A | 1187096 | NULL | NULL | | BTREE | | 
	| ad_visit_history | 1 | ad_code | 1 | ad_code | A | 46 | NULL | NULL | YES | BTREE | | 
	| ad_visit_history | 1 | unique_id | 1 | unique_id | A | 1187096 | NULL | NULL | YES | BTREE | | 
	| ad_visit_history | 1 | ad_code_ind | 1 | ad_code | A | 46 | NULL | NULL | YES | BTREE | | 
	| ad_visit_history | 1 | from_page_url_ind | 1 | from_page_url | A | 30438 | NULL | NULL | YES | BTREE | | 
	| ad_visit_history | 1 | ip_ind | 1 | ip | A | 593548 | NULL | NULL | YES | BTREE | | 
	| ad_visit_history | 1 | port_ind | 1 | port | A | 65949 | NULL | NULL | YES | BTREE | | 
	| ad_visit_history | 1 | session_id_ind | 1 | session_id | A | 1187096 | NULL | NULL | YES | BTREE | | 
	+------------------+------------+-------------------+--------------+---------------+-----------+-------------+----------+--------+------+------------+---------+ 
	8 rows in set (0.28 sec)

**索引信息中的列的信息说明**

Table :表的名称  
Non_unique:如果索引不能包括重复词，则为0。如果可以，则为1  
Key_name:索引的名称  
Seq_in_index:索引中的列序列号，从1开始  
Column_name:列名称  
Collation:列以什么方式存储在索引中。在MySQLSHOW INDEX语法中，有值’A’（升序）或NULL（无分类）   
Cardinality:索引中唯一值的数目的估计值。通过运行ANALYZE TABLE或myisamchk -a可以更新。基数根据被存储为整数的统计数据来计数，所以即使对于小型表，该值也没有必要是精确的。基数越大，当进行联合时，MySQL使用该索引的机会就越大  
Sub_part:如果列只是被部分地编入索引，则为被编入索引的字符的数目。如果整列被编入索引，则为NULL  
Packed:指示关键字如何被压缩。如果没有被压缩，则为NULL  
Null:如果列含有NULL，则含有YES。如果没有，则为空  
Index_type：存储索引数据结构方法（BTREE, FULLTEXT, HASH, RTREE）  

**二、删除一半数据**

	mysql> delete from ad_visit_history where id>598000; //删除一半数据 
	Query OK, 589096 rows affected (4 min 28.06 sec)

	[root@ www.linuxidc.com test1]# ls |grep visit |xargs -i du {} //相对应的MYD，MYI文件大小没有变化 
	382020 ad_visit_history.MYD 
	127116 ad_visit_history.MYI 
	12 ad_visit_history.frm

按常规思想来说，如果在数据库中删除了一半数据后，相对应的.MYD,.MYI文件也应当变为之前的一半。但是删除一半数据后，.MYD.MYI尽然连1KB都没有减少，这是多么的可怕啊  

我们在来看一看，索引信息  

	mysql> show index from ad_visit_history; 
	+------------------+------------+-------------------+--------------+---------------+-----------+-------------+----------+--------+------+------------+---------+ 
	| Table | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | 
	+------------------+------------+-------------------+--------------+---------------+-----------+-------------+----------+--------+------+------------+---------+ 
	| ad_visit_history | 0 | PRIMARY | 1 | id | A | 598000 | NULL | NULL | | BTREE | | 
	| ad_visit_history | 1 | ad_code | 1 | ad_code | A | 23 | NULL | NULL | YES | BTREE | | 
	| ad_visit_history | 1 | unique_id | 1 | unique_id | A | 598000 | NULL | NULL | YES | BTREE | | 
	| ad_visit_history | 1 | ad_code_ind | 1 | ad_code | A | 23 | NULL | NULL | YES | BTREE | | 
	| ad_visit_history | 1 | from_page_url_ind | 1 | from_page_url | A | 15333 | NULL | NULL | YES | BTREE | | 
	| ad_visit_history | 1 | ip_ind | 1 | ip | A | 299000 | NULL | NULL | YES | BTREE | | 
	| ad_visit_history | 1 | port_ind | 1 | port | A | 33222 | NULL | NULL | YES | BTREE | | 
	| ad_visit_history | 1 | session_id_ind | 1 | session_id | A | 598000 | NULL | NULL | YES | BTREE | | 
	+------------------+------------+-------------------+--------------+---------------+-----------+-------------+----------+--------+------+------------+---------+ 
	8 rows in set (0.00 sec)

对比一下，这次索引查询和上次索引查询，里面的数据信息基本上是上次一次的一半，这点还是合乎常理。

 

**三、用optimize table来优化一下**

	mysql> optimize table ad_visit_history; //删除数据后的优化 
	+------------------------+----------+----------+----------+ 
	| Table | Op | Msg_type | Msg_text | 
	+------------------------+----------+----------+----------+ 
	| test1.ad_visit_history | optimize | status | OK | 
	+------------------------+----------+----------+----------+ 
	1 row in set (1 min 21.05 sec)

1、查看一下.MYD,.MYI文件的大小

	[root@ www.linuxidc.com test1]# ls |grep visit |xargs -i du {} 
	182080 ad_visit_history.MYD //数据文件差不多为优化前的一半 
	66024 ad_visit_history.MYI //索引文件也一样，差不多是优化前的一半 
	12 ad_visit_history.frm

2，查看一下索引信息

	mysql> show index from ad_visit_history; 
	+------------------+------------+-------------------+--------------+---------------+-----------+-------------+----------+--------+------+------------+---------+ 
	| Table | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | 
	+------------------+------------+-------------------+--------------+---------------+-----------+-------------+----------+--------+------+------------+---------+ 
	| ad_visit_history | 0 | PRIMARY | 1 | id | A | 598000 | NULL | NULL | | BTREE | | 
	| ad_visit_history | 1 | ad_code | 1 | ad_code | A | 42 | NULL | NULL | YES | BTREE | | 
	| ad_visit_history | 1 | unique_id | 1 | unique_id | A | 598000 | NULL | NULL | YES | BTREE | | 
	| ad_visit_history | 1 | ad_code_ind | 1 | ad_code | A | 42 | NULL | NULL | YES | BTREE | | 
	| ad_visit_history | 1 | from_page_url_ind | 1 | from_page_url | A | 24916 | NULL | NULL | YES | BTREE | | 
	| ad_visit_history | 1 | ip_ind | 1 | ip | A | 598000 | NULL | NULL | YES | BTREE | | 
	| ad_visit_history | 1 | port_ind | 1 | port | A | 59800 | NULL | NULL | YES | BTREE | | 
	| ad_visit_history | 1 | session_id_ind | 1 | session_id | A | 598000 | NULL | NULL | YES | BTREE | | 
	+------------------+------------+-------------------+--------------+---------------+-----------+-------------+----------+--------+------+------------+---------+ 
	8 rows in set (0.00 sec)

从以上数据我们可以得出，ad_code，ad_code_ind，from_page_url_ind等索引机会差不多都提高了85％，这样效率提高了好多

四、小结

结合mysql官方网站的信息，个人是这样理解的。当你删除数据时，mysql并不会回收，被已删除数据的占据的存储空间，以及索引位。而是空在那里，而是等待新的数据来弥补这个空缺，这样就有一个缺少，如果一时半会，没有数据来填补这个空缺，那这样就太浪费资源了。所以对于写比较频烦的表，要定期进行optimize，一个月一次，看实际情况而定了



   
