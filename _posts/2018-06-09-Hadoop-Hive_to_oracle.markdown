---
layout:     post
title:      "导出hive的表到oracle"
subtitle:   "Hive几种导入导出数据方式"
date:       2018-06-09
author:     "kikyoar"
header-img: "img/post-bg-hadoop-version.jpg"
tags:
    - Hadoop
---   

## hive 数据导入表的方式  

**1、本地导入**  
创建学生表， 将”\t”作为分割  

	CREATE TABLE student
	     (id int ,
	     name string,
	     sex string
	     )
	     row format delimited fields terminated by '\t';  

本地文件data.txt如下:(\t分割）   

	1       zhangsan        man
	2       lisi    man
	3       liwu    woman  
	
追加：

	load data local inpath 'data.txt' into table student;  
	
覆盖:  

	load data local inpath 'data.txt' overwrite into table student;  
	
**2、从hdfs导入**  

追加：  

	load data inpath '/user/hive/warehouse/studentdata.txt' into table student2;   

覆盖:  

	load data inpath '/user/hive/warehouse/studentdata.txt' overwrite into table student2;  
	
**和本地的语句少个local。导入数据到表要注意 分割符必须一致， 不然导入后为null**  

**3、使用insert从别的表导入数据**  

追加：

	insert into table 表1 select * from 表2 [where条件]  
	
覆盖：  

	insert verwrite table 表1 select * from 表2 [where条件]  
	
**4、关系型数据库导入数据**  

	sqoop import –connect jdbc:mysql://IP:3306/dbname –username xxx –password xxx –table table1 –hive-import –hive-overwrite –create-hive-table –hive-table dbname.table1	

## hive 数据导出表的方式  

**1、拷贝文件**
 

如果数据文件恰好是用户需要的格式，那么只需要拷贝文件或文件夹就可以  

	hadoop fs –cp source_path target_path

**2、导出到本地文件系统**

**不能使用insert into local directory来导出数据，会报错**
**只能使用insert overwrite local directory来导出数据**

hive0.11版本之前，只能使用默认分隔符\^A(ascii码是\00001)
	
	insert overwrite local directory '/home/sopdm/wrk'
	
	select id,name,tel,age from sopdm.wyp; 

hive0.11版本之后，可以指定分隔符

	insert overwrite local directory '/home/sopdm/wrk'
	
	row format delimited
	
	fields terminated by ','
	
	select id,name,tel,age from sopdm.wyp;  

 导出数据到多个输出文件夹

	from employees se
	
	insert overwrite local directory ‘/tmp/or_employees’
	
	     select * se where se.cty=’US’ and se.st=’OR’
	
	insert overwrite local directory ‘/tmp/ca_employees’
	
	     select * se where se.cty=’US’ and se.st=’CA’

**3、导出到HDFS**

比导出文件到本地文件系统少了一个local

	insert overwritedirectory '/home/sopdm/wrk'
	
	select id,name,tel,age from sopdm.wyp;

hive0.11版本之后，可以指定分隔符

	insert overwritedirectory '/home/sopdm/wrk'
	
	row format delimited
	
	fields terminated by ','
	
	select id,name,tel,age from sopdm.wyp;

**4、导出到hive的另一张表**

	insert into table sopdm.wyp2
	
	partition(age='25')
	
	select id,name,tel,age from sopdm.wyp;

**5、使用hive的-e和-f参数命令导出数据**

使用hive的-e参数

	hive –e “select * from wyp” >> /local/wyp.txt

使用hive的-f参数, wyp.hql中为hql语句

	hive –f wyp.hql >> /local/wyp2.txt

6、导出到关系型数据库

	Sqoop export –connect jdbc:mysql://127.0.0.1:3306/dbname –username mysql(mysql用户名) –password 123456(密码) –table student(mysql上的表) –hcatalog-database sopdm(hive上的schema) –hcatalog-table student(hive上的表)  
	
## 实例分析（hive数据库的表导入到oracle）  

**思路**  

**1、首先hive里面的数据如果直接导出**  

	insert overwrite local directory '/root/export_hive’
	SELECT procedure_starttime, imsi, imei, seci, enb_ue_s1ap_id, mme_ue_s1ap_id, tmsi, procedure_type, endtime, starttime, sdate, city FROM daily.o_lte_s1mme_standard WHERE dh="2018060622" LIMIT 10;

这样直接导出出现的问题是当数据导入oracle时会直接导入到第一个字段中，因为数据中的列与列之间的分隔符是\^A(ascii码是\00001，为了避免插入oracle表存在问题，采用如下办法:

**2、hive导出数据用分隔符,（不用制表符的原因是导入的时候即使标明字段以'\t'结尾，也会出现上述混到一行的情况，所以最好使用其他分隔符号，诸如\*之类）**  

	insert overwrite local directory '/root/export_hive'
	row format delimited
	fields terminated by ','
	SELECT procedure_starttime, imsi, imei, seci, enb_ue_s1ap_id, mme_ue_s1ap_id, tmsi, procedure_type, endtime, starttime, sdate, city FROM daily.o_lte_s1mme_standard WHERE dh="2018060622";   
	
![hive_oracle_01](http://kikyoar.com/img/hive_oracle_01.png)  

等待导完后，在/root/export_hive文件夹中  

![hive_oracle_02](http://kikyoar.com/img/hive_oracle_02.png)   

全部是分散文件，使用命令整合为一个文件
	
	cat *_0 >> o_lte_s1mme_standard

将其拷贝到oracle数据库/home/oracle目录下

**3、在oracle数据库中创建表**

	CREATE TABLE o_lte_s1mme_standard （procedure_starttime varchar2(30),
	imsi NUMBER(19),imei VARCHAR2(16),seci NUMBER(19),enb_ue_s1ap_id NUMBER(19),mme_ue_s1ap_id NUMBER(19),m_tmsi NUMBER(19),procedure_type NUMBER(5),endtime varchar2(30),starttime varchar2(30),sdate varchar2(30),city varchar2(30));   
	
**所有TIME格式采用varchar2(30)，而不采用timestap(6)，原因是由于hive中的数据格式如：2018-06-06 22:30:18.378，在oracle里面不支持，上传数据的时候会报错**  

**4、在/home/oracle创建文件**  

	[root@oracle oracle]# vi input.ctl
	
	load data
	infile '/home/oracle/o_lte_s1mme_standard.csv'
	append into table o_lte_s1mme_standard
	fields terminated by ','
	OPTIONALLY ENCLOSED BY '"'
	TRAILING NULLCOLS
	(
	procedure_starttime,
	imsi,
	imei,
	seci,
	enb_ue_s1ap_id,
	mme_ue_s1ap_id,
	m_tmsi,
	procedure_type,
	endtime,
	starttime,
	sdate,
	city
	)  
	
**以上表均要按照创建表的字段顺序写入**  

	[root@oracle oracle]#sqlldr  userid=c数据库用户名/数据库密码 control =/home/oracle/input.ctl   
	
数据导入完成

 
