---
layout:     post
title:      "Hive数据类型"
subtitle:   "Hive的几种数据类型解析"
date:       2018-06-06
author:     "kikyoar"
header-img: "img/post-bg-hadoop-version.jpg"
tags:
    - Hadoop
---  

**Hive既有大多数关系数据库中的基本类型，又有集合这种复杂类型**  


**单精度和双精度的区别**   

*含义：表明单精度和双精度精确的范围不一样，单精度，也即float，一般在计算机中存储占用4字节，也32位，有效位数为7位；双精度（double）在计算机中存储占用8字节，64位，有效位数为16位*  

*原因：不管float还是double 在计算机上的存储都遵循IEEE规范，使用二进制科学计数法，都包含三个部分：符号位，指数位和尾数部分。其中float的符号位，指数位，尾数部分分别为1,  8,  23. 双精度分别为1,  11,  52。精度主要取决于尾数部分的位数，float为23位，除去全部为0的情况以外，最小为2的-23次方，约等于1.19乘以10的-7次方，所以float小数部分只能精确到后面6位，加上小数点前的一位，即有效数字为7位。  类似，double 尾数部分52位，最小为2的-52次方，约为2.22乘以10的-16次方，所以精确到小数点后15位，有效位数为16位*

## 基本类型   

数据类型 | 大小  | 范围  |  示例
:--------:|:--------|:--------:|:-------:
TINYINT|	1byte|	-128 ~ 127	|100Y
SMALLINT|	2byte|	-32,768 ~ 32,767|	100S
INT/INTEGER|	4byte|	-2,147,483,648 ~ 2,147,483,647 | 100
BIGINT	|8byte|	-9,223,372,036,854,775,808 ~ 9,223,372,036,854,775,807|100L
FLOAT	|4byte|	单精度浮点数	|3.1415926
DOUBLE	|8byte	|双精度浮点数	|3.1415926
DECIMAL|	-	|高精度浮点数	|DECIMAL(9,8)
BOOLEAN|	-	|布尔型，TRUE/FALSE|	true
BINARY	|-	|二进制类型|	-


- **整数类型**  
默认情况下，整数型为INT型，当数字大于INT型的范围时，会自动解释执行为BIGINT，或者使用以下后缀进行说明  

	类型 |	后缀	| 例子  
	:--------:|:--------|:------:  
	TINYINT|	Y	|100Y  
	SMALLINT	|S	|100S
	BIGINT	|L	|100L  

	**-2,147,483,648 ~ 2,147,483,647之间的整数类型默认是INT型，除非指定了格式100Y、100S、100L会自动转换为TINYINT、SMALLINT、BIGINT**   
	
- **浮点数类型**  
	Hive的小数型是基于Java BigDecimal做的， BigDecimal在java中用于表示任意精度的小数类型。所有常规数字运算（例如+， - ，*，/）和相关的UDFs（例如Floor，Ceil，Round等等）都使用和支持Decimal。你可以将Decimal和其他数值型互相转换，且Decimal支持科学计数法和非科学计数法。因此，无论您的数据集是否包含如4.004E + 3（科学记数法）或4004（非科学记数法）或两者的组合的数据，可以使用Decimal。从Hive 0.13开始，用户可以使用DECIMAL(precision, scale) 语法在创建表时来定义Decimal数据类型的precision(精度)和scale（规模）。 如果未指定precision，则默认为10。如果未指定scale，它将默认为0（无小数位）   

		CREATE TABLE foo ( 
		a DECIMAL, – Defaults to decimal(10,0) 
		b DECIMAL(9, 7) 
		)
	大于BIGINT的数值，需要使用BD后缀以及Decimal(38,0)来处理，例：

		select CAST(18446744073709001000BD AS DECIMAL(38,0)) from my_table limit 1;

	Decimal在Hive 0.12.0 and 0.13.0之间是不兼容的，故0.12前的版本需要迁移才可继续使用  
	
- **字符串类型**   

	**String**  
	string类型可以用单引号（'）或双引号（"）定义，Hive在string中使用C-style  

	**Varchar**  
	varchar类型由长度定义，范围为1-65355，如果存入的字符串长度超过了定义的长度，超出部分会被截断。尾部的空格也会作为字符串的一部分，影响字符串的比较  

	**Char**  
	char是固定长度的，最大长度255，而且尾部的空格不影响字符串的比较。


	三种类型对尾部空格的区别，参考如下例子，每个字段都插入同样的字符并且在尾部有不同的空格
	
		hive> create table char_a(
		    > c1 char(4),
		    > c2 char(5),
		    > str1 string,
		    > str2 string,
		    > var1 varchar(4),
		    > var2 varchar(5)
		    > )
		    > ;
		    
		hive> insert into char_a values('ccc ','ccc  ','ccc ','ccc  ','ccc ','ccc  ');
		
		hive> select c1=c2,str1=str2,var1=var2 from char_a;
		
		OK
		true    false   false
		Time taken: 1.101 seconds, Fetched: 1 row(s)   

## 日期与时间戳  

**Timestamps**   
支持传统的UNIX时间戳和可选的纳秒精度   
支持的转化：   

- 整数数字类型：以秒为单位解释为UNIX时间戳    
- 浮点数值类型：以秒为单位解释为UNIX时间戳，带小数精度   
- 字符串：符合JDBC java.sql.Timestamp格式“YYYY-MM-DD HH：MM：SS.fffffffff”（9位小数位精度）   
时间戳被解释为无时间的，并被存储为从Unix纪元的偏移量  

所有现有的日期时间UDFs（月，日，年，小时等）都使用TIMESTAMP数据类型  
 
Text files中的时间戳必须使用格式yyyy-mm-dd hh:mm:ss [.f …]。 如果它们是另一种格式，请将它们声明为适当的类型（INT，FLOAT，STRING等），并使用UDF将它们转换为时间戳  
 
在表级别上，可以通过向SerDe属性”timestamp.formats”（自版本1.2.0 with HIVE-9298）提供格式来支持备选时间戳格式。 例如，yyyy-MM-dd’T’HH:mm:ss.SSS，yyyy-MM-dd’T’HH:mm:ss   

**Dates**    
DATE值描述特定的年/月/日，格式为YYYY-MM-DD。 例如，DATE’2013-01-01’。 日期类型没有时间组件。 Date类型支持的值范围是0000-01-01到9999-12-31，这取决于Java Date类型的原始支持   

Date types只能在Date, Timestamp, or String types之间转换    


转换类型|	结果
:--------:|:--------
cast(date as date)|	Same date value
cast(date as string)|	The year/month/day represented by the Date is formatted as a string in the form ‘YYYY-MM-DD’
cast(date as timestamp)|	A timestamp value is generated corresponding to midnight of the year/month/day of the date value, based on the local timezone
cast(string as date)|	If the string is in the form ‘YYYY-MM-DD’, then a date value corresponding to that year/month/day is returned. If the string value does not match this formate, then NULL is returned
cast(timestamp as date)	|The year/month/day of the timestamp is determined, based on the local timezone, and returned as a date value  

**Intervals**  

后续补充  

## 复杂类型  

**STRUCT**  
类似于C、C#语言，Hive中定义的struct类型也可以使用点来访问。从文件加载数据时，文件里的数据分隔符要和建表指定的一致  

	CREATE TABLE IF NOT EXISTS person_1 (id int,info struct<name:string,country:string>)  
	ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' 
	COLLECTION ITEMS TERMINATED BY ':' 
	STORED AS TEXTFILE;
	
	创建一个文本文件test_struct.txt
	1,'dd':'jp'
	2,'ee':'cn'
	3,'gg':'jp'
	4,'ff':'cn'
	5,'tt':'jp'
	
	导入数据
	LOAD DATA LOCAL INPATH '/data/test_struct.txt' OVERWRITE INTO TABLE person_1;
	
	查询数据
	hive> select * from person_1;
	OK
	1   {"name":"'dd'","country":"'jp'"}
	2   {"name":"'ee'","country":"'cn'"}
	3   {"name":"'gg'","country":"'jp'"}
	4   {"name":"'ff'","country":"'cn'"}
	5   {"name":"'tt'","country":"'jp'"}
	Time taken: 0.046 seconds, Fetched: 5 row(s)
	
	hive> select id,info.name,info.country from person_1 where info.name='dd';
	OK
	1   dd  jp
	Time taken: 1.166 seconds, Fetched: 1 row(s)

**ARRAY**  
ARRAY表示一组相同数据类型的集合，下标从零开始，可以用下标访问  

	CREATE TABLE IF NOT EXISTS array_1 (id int,name array<STRING>)
	ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' 
	COLLECTION ITEMS TERMINATED BY ':' 
	STORED AS TEXTFILE;
	
	导入数据
	LOAD DATA LOCAL INPATH '/data/test_array.txt' OVERWRITE INTO TABLE array_1;
	
	查询数据
	hive> select * from array_1;
	OK
	1   ["dd","jp"]
	2   ["ee","cn"]
	3   ["gg","jp"]
	4   ["ff","cn"]
	5   ["tt","jp"]
	Time taken: 0.041 seconds, Fetched: 5 row(s)
	
	hive> select id,name[0],name[1] from array_1 where name[1]='cn';
	OK
	2   ee  cn
	4   ff  cn
	Time taken: 1.124 seconds, Fetched: 2 row(s)

**MAP**  
MAP是一组键值对的组合，可以通过KEY访问VALUE，键值之间同样要在创建表时指定分隔符  

	CREATE TABLE IF NOT EXISTS map_1 (id int,name map<STRING,STRING>)
	ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' 
	COLLECTION ITEMS TERMINATED BY ':' 
	MAP KEYS TERMINATED BY ':'
	STORED AS TEXTFILE;
	
	加载数据
	LOAD DATA LOCAL INPATH '/data/test_map.txt' OVERWRITE INTO TABLE map_1;
	
	查询数据
	hive> select * from map_1;
	OK
	1   {"name":"dd","country":"jp"}
	2   {"name":"ee","country":"cn"}
	3   {"name":"gg","country":"jp"}
	4   {"name":"ff","country":"cn"}
	5   {"name":"tt","country":"jp"}
	Time taken: 0.038 seconds, Fetched: 5 row(s)
	
	select id,info['name'],info['country'] from map_1 where info['country']='cn';
	OK
	2   ee  cn
	4   ff  cn
	Time taken: 1.088 seconds, Fetched: 2 row(s)

**UINON TYPES**     
Hive除了支持STRUCT、ARRAY、MAP这些原生集合类型，还支持集合的组合，不支持集合里再组合多个集合。简单示例MAP嵌套ARRAY，手动设置集合格式的数据非常麻烦，建议采用INSERT INTO SELECT 形式构造数据再插入UNION表      

	创建DUAL表，插入一条记录，用于生成数据
	create table dual(d string);
	insert into dual values('X');
	
	创建UNION表
	CREATE TABLE IF NOT EXISTS uniontype_1 
	(
	id int,
	info map<STRING,array<STRING>>
	)
	ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' 
	COLLECTION ITEMS TERMINATED BY '-'
	MAP KEYS TERMINATED BY ':'
	STORED AS TEXTFILE;
	
	插入数据
	insert overwrite table uniontype_1
	select 1 as id,map('english',array(99,21,33)) as info from dual
	union all
	select 2 as id,map('english',array(44,33,76)) as info from dual
	union all
	select 3 as id,map('english',array(76,88,66)) as info from dual;
	
	查询数据
	hive> select * from uniontype_1;
	OK
	3   {"german":[76,88,66]}
	2   {"chinese":[44,33,76]}
	1   {"english":[99,21,33]}
	Time taken: 0.033 seconds, Fetched: 3 row(s)
	
	hive> select * from uniontype_1 where info['english'][2]>30;
	OK
	1   {"english":[99,21,33]}
	Time taken: 1.08 seconds, Fetched: 1 row(s)






















