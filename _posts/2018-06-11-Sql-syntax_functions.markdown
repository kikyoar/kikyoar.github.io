---
layout:     post
title:      "sql基础语法和函数"
subtitle:   "sql语法和函数解析"
date:       2018-06-11
author:     "kikyoar"
header-img: "img/post-bg-linux-version.jpg"
tags:
    - database
---   

## sql基础语法  

**所有实例均适合用于postgresql**

**1、SQL SELECT 语句**   

	SELECT 列名称 FROM 表名称 
	select * from persons;   
		
**符号 \* 取代列的名称,星号（\*）是选取所有列的快捷方式**  

**2、SQL SELECT DISTINCT 语句**  

**关键词 DISTINCT 用于返回唯一不同的值**  

	SELECT DISTINCT 列名称 FROM 表名称  
	SELECT DISTINCT Company FROM Orders;  
		
**3、SQL WHERE 子句**   

**WHERE 子句用于规定选择的标准**  
**=、<>、>、<、>=、<=、BETWEEN、LIKE可在where子句中使用**   

	SELECT 列名称 FROM 表名称 WHERE 列 运算符 值
	SELECT * FROM Persons WHERE City='Beijing';  

**SQL 使用单引号来环绕文本值（大部分数据库系统也接受双引号）。如果是数值，请不要使用引号**  

**4、SQL AND & OR 运算符**  

**AND 和 OR 运算符用于基于一个以上的条件对记录进行过滤**   
**AND 和 OR 可在 WHERE 子语句中把两个或多个条件结合起来。如果第一个条件和第二个条件都成立，则 AND 运算符显示一条记录。如果第一个条件和第二个条件中只要有一个成立，则 OR 运算符显示一条记录**   

	# 使用 AND 来显示所有姓为 "Carter" 并且名为 "Thomas" 的人 :
	SELECT * FROM Persons WHERE FirstName='Thomas' AND LastName='Carter';
	# 使用 OR 来显示所有姓为 "Carter" 或者名为 "Thomas" 的人 :
	SELECT * FROM Persons WHERE firstname='Thomas' OR lastname='Carter';
	# 我们也可以把 AND 和 OR 结合起来（使用圆括号来组成复杂的表达式）:
	SELECT * FROM Persons WHERE (FirstName='Thomas' OR FirstName='William') AND LastName='Carter';

**5、SQL ORDER BY 子句**  

**ORDER BY 语句用于对结果集进行排序**  
**ORDER BY 语句用于根据指定的列对结果集进行排序。ORDER BY 语句默认按照升序对记录进行排序。如果您希望按照降序对记录进行排序，可以使用 DESC 关键字。**  

	SELECT Company, OrderNumber FROM Orders ORDER BY Company;
	SELECT Company, OrderNumber FROM Orders ORDER BY Company DESC;  
	
**6、INSERT INTO 语句**   

**INSERT INTO 语句用于向表格中插入新的行**   

	INSERT INTO 表名称 VALUES (值1, 值2,....)
	INSERT INTO table_name (列1, 列2,...) VALUES (值1, 值2,....)
	INSERT INTO Persons VALUES ('Gates', 'Bill', 'Xuanwumen 10', 'Beijing')
	INSERT INTO Persons (LastName, Address) VALUES ('Wilson', 'Champs-Elysees')  
	
**7、SQL UPDATE 语句**  

**Update 语句用于修改表中的数据**  

	UPDATE 表名称 SET 列名称 = 新值 WHERE 列名称 = 某值
	UPDATE Person SET FirstName = 'Fred' WHERE LastName = 'Wilson';
	UPDATE Person SET Address = 'Zhongshan 23', City = 'Nanjing' WHERE LastName = 'Wilson';

**8、SQL DELETE 语句**  

**DELETE 语句用于删除表中的行**  

	DELETE FROM 表名称 WHERE 列名称 = 值
	DELETE FROM Person WHERE LastName = 'Wilson' 

**可以在不删除表的情况下删除所有的行。这意味着表的结构、属性和索引都是完整的**   

	DELETE FROM table_name
	DELETE * FROM table_name  
	
**9、SQL TOP 子句**  

**TOP 子句用于规定要返回的记录的数目。对于拥有数千条记录的大型表来说，TOP 子句是非常有用的**   
**并非所有的数据库系统都支持 TOP 子句**   

	SELECT column_name(s) FROM table_name LIMIT number
	SELECT * FROM Persons LIMIT 5  
	
**10、SQL LIKE 操作符**  

**LIKE 操作符用于在 WHERE 子句中搜索列中的指定模式**  

	SELECT column_name(s) FROM table_name WHERE column_name LIKE pattern  
	SELECT * FROM Persons WHERE City LIKE 'N%'  

**"%" 可用于定义通配符（模式中缺少的字母）**  

**11、SQL 通配符**  

**在搜索数据库中的数据时，SQL 通配符可以替代一个或多个字符。SQL 通配符必须与 LIKE 运算符一起使用**  


|通配符|	描述|
:-----:|:------:
% |	替代一个或多个字符
_	| 仅替代一个字符
[charlist]|	字符列中的任何单一字符
[\^charlist] 或者 [!charlist] | 不在字符列中的任何单一字符   

**12、SQL IN 操作符**  

**IN 操作符允许我们在 WHERE 子句中规定多个值**  

	SELECT column_name(s) FROM table_name WHERE column_name IN (value1,value2,...)  
	SELECT * FROM Persons WHERE LastName IN ('Adams','Carter');  
	
**13、SQL BETWEEN 操作符**  

**BETWEEN 操作符在 WHERE 子句中使用，作用是选取介于两个值之间的数据范围**   
**操作符 BETWEEN ... AND 会选取介于两个值之间的数据范围。这些值可以是数值、文本或者日期**  

	SELECT column_name(s) FROM table_name WHERE column_name BETWEEN value1 AND value2
	SELECT * FROM Persons WHERE LastName BETWEEN 'Adams' AND 'Carter'
	SELECT * FROM Persons WHERE LastName NOT BETWEEN 'Adams' AND 'Carter'  
	
**14、SQL Alias（别名）**  

**通过使用 SQL，可以为列名称和表名称指定别名（Alias）**   
**表的 SQL Alias 语法**  

	SELECT column_name(s) FROM table_name AS alias_name 
	SELECT po.OrderID, p.LastName, p.FirstName FROM Persons AS p, Product_Orders AS po WHERE p.LastName='Adams' AND p.FirstName='John' 
	
**列的 SQL Alias 语法**  

	SELECT column_name AS alias_name FROM table_name  
	SELECT LastName AS Family, FirstName AS Name FROM Persons   
	
**15、SQL JOIN**  

**SQL join 用于根据两个或多个表中的列之间的关系，从这些表中查询数据**  
**有时为了得到完整的结果，我们需要从两个或更多的表中获取结果。我们就需要执行 join。数据库中的表可通过键将彼此联系起来。主键（Primary Key）是一个列，在这个列中的每一行的值都是唯一的。在表中，每个主键的值都是唯一的。这样做的目的是在不重复每个表中的所有数据的情况下，把表间的数据交叉捆绑在一起**  

- INNER JOIN: 如果表中有至少一个匹配，则返回行，INNER JOIN 与 JOIN 是相同的

		SELECT column_name(s)
		FROM table_name1
		INNER JOIN table_name2 
		ON table_name1.column_name=table_name2.column_name
		
		# 我们希望列出所有人的定购
		SELECT Persons.LastName, Persons.FirstName, Orders.OrderNo
		FROM Persons
		INNER JOIN Orders
		ON Persons.Id_P=Orders.Id_P
		ORDER BY Persons.LastName

**INNER JOIN 关键字在表中存在至少一个匹配时返回行。如果 "Persons" 中的行在 "Orders" 中没有匹配，就不会列出这些行**  

- LEFT JOIN:会从左表 (table_name1) 那里返回所有的行，即使在右表 (table_name2) 中没有匹配的行  

		SELECT column_name(s)
		FROM table_name1
		LEFT JOIN table_name2 
		ON table_name1.column_name=table_name2.column_name  
		
		
		#希望列出所有的人，以及他们的定购 - 如果有的话
		SELECT Persons.LastName, Persons.FirstName, Orders.OrderNo
		FROM Persons
		LEFT JOIN Orders
		ON Persons.Id_P=Orders.Id_P
		ORDER BY Persons.LastName		

**LEFT JOIN 关键字会从左表 (Persons) 那里返回所有的行，即使在右表 (Orders) 中没有匹配的行**

- RIGHT JOIN:会从右表 (table_name2) 那里返回所有的行，即使在左表(table_name1) 中没有匹配的行   

		SELECT column_name(s)
		FROM table_name1
		RIGHT JOIN table_name2 
		ON table_name1.column_name=table_name2.column_name  
		
		# 我们希望列出所有的定单，以及定购它们的人 - 如果有的话  
		SELECT Persons.LastName, Persons.FirstName, Orders.OrderNo
		FROM Persons
		RIGHT JOIN Orders
		ON Persons.Id_P=Orders.Id_P
		ORDER BY Persons.LastName  
		
**RIGHT JOIN 关键字会从右表 (Orders) 那里返回所有的行，即使在左表 (Persons) 中没有匹配的行**

- FULL JOIN: 只要其中一个表中存在匹配，就返回行  

		SELECT column_name(s)
		FROM table_name1
		FULL JOIN table_name2 
		ON table_name1.column_name=table_name2.column_name   

		# 希望列出所有的人，以及他们的定单，以及所有的定单，以及定购它们的人
		SELECT Persons.LastName, Persons.FirstName, Orders.OrderNo
		FROM Persons
		FULL JOIN Orders
		ON Persons.Id_P=Orders.Id_P
		ORDER BY Persons.LastName  
		
**FULL JOIN 关键字会从左表 (Persons) 和右表 (Orders) 那里返回所有的行。如果 "Persons" 中的行在表 "Orders" 中没有匹配，或者如果 "Orders" 中的行在表 "Persons" 中没有匹配，这些行同样会列出**   

**16、SQL UNION 和 UNION ALL 操作符**   

**UNION 操作符用于合并两个或多个 SELECT 语句的结果集**  
**请注意，UNION 内部的 SELECT 语句必须拥有相同数量的列。列也必须拥有相似的数据类型。同时，每条 SELECT 语句中的列的顺序必须相同**  

**SQL UNION 语法**

	SELECT column_name(s) FROM table_name1
	UNION
	SELECT column_name(s) FROM table_name2  

	# 列出所有在中国和美国的不同的雇员名
	SELECT E_Name FROM Employees_China
	UNION
	SELECT E_Name FROM Employees_USA
	
**UNION 操作符选取不同的值。如果允许重复的值，请使用 UNION ALL** 
	
**SQL UNION ALL 语法**  

	SELECT column_name(s) FROM table_name1
	UNION ALL
	SELECT column_name(s) FROM table_name2  
	
	# 列出在中国和美国的所有的雇员  
	SELECT E_Name FROM Employees_China
	UNION ALL
	SELECT E_Name FROM Employees_USA  

**17、SQL SELECT INTO 语句**  

**SQL SELECT INTO 语句可用于创建表的备份复件**  
**SELECT INTO 语句从一个表中选取数据，然后把数据插入另一个表中。SELECT INTO 语句常用于创建表的备份复件或者用于对记录进行存档**  

	SELECT column_name(s)
	INTO new_table_name [IN externaldatabase] 
	FROM old_tablename  
	
**下面的例子会制作 "Persons" 表的备份复件：**  

	SELECT *
	INTO Persons_backup
	FROM Persons

**IN 子句可用于向另一个数据库中拷贝表，postgresql不支持：**  

	SELECT *
	INTO Persons IN 'Backup.mdb'
	FROM Persons  

**希望拷贝某些域，可以在 SELECT 语句后列出这些域：**  

	SELECT LastName,FirstName
	INTO Persons_backup
	FROM Persons

**带有 WHERE 子句：**

	SELECT LastName,Firstname
	INTO Persons_backup
	FROM Persons
	WHERE City='Beijing'  
	
**被连接的表：**  

	SELECT Persons.LastName,Orders.OrderNo
	INTO Persons_Order_Backup
	FROM Persons
	INNER JOIN Orders
	ON Persons.Id_P=Orders.Id_P  
	
**18、SQL CREATE DATABASE 语句**  

**CREATE DATABASE 用于创建数据库**  

	CREATE DATABASE database_name
	CREATE DATABASE my_db  
	
**19、SQL CREATE TABLE 语句**  

**CREATE TABLE 语句用于创建数据库中的表**  

	CREATE TABLE 表名称
	(
	列名称1 数据类型,
	列名称2 数据类型,
	列名称3 数据类型,
	....
	)  
	
|数据类型|描述|
| :------: | :------: |
| integer(size) int(size) smallint(size) tinyint(size) | 仅容纳整数，在括号内规定数字的最大位数 |
| decimal(size,d) numeric(size,d)  | 容纳带有小数的数字，"size" 规定数字的最大位数。"d" 规定小数点右侧的最大位数 |
| char(size) | 容纳固定长度的字符串（可容纳字母、数字以及特殊字符），在括号中规定字符串的长度 |
| varchar(size) | 容纳可变长度的字符串（可容纳字母、数字以及特殊的字符），在括号中规定字符串的最大长度 |
| date(yyyymmdd) | 容纳日期 |  

	CREATE TABLE Persons
	(
	Id_P int,
	LastName varchar(255),
	FirstName varchar(255),
	Address varchar(255),
	City varchar(255)
	)   
	
**20、SQL 约束 (Constraints)**  

**约束用于限制加入表的数据的类型**  
**可以在创建表时规定约束（通过 CREATE TABLE 语句），或者在表创建之后也可以（通过 ALTER TABLE 语句**   

- SQL NOT NULL 约束  

	**NOT NULL 约束强制列不接受 NULL 值**  
	**NOT NULL 约束强制字段始终包含值。这意味着，如果不向字段添加值，就无法插入新记录或者更新记录**  
	
	
		CREATE TABLE Persons
		(
		Id_P int NOT NULL,
		LastName varchar(255) NOT NULL,
		FirstName varchar(255),
		Address varchar(255),
		City varchar(255)
		)
		
		#更改数据类型和设置部位空   
		alter table persons_order_backup alter lastname type varchar(30);
		alter table persons_order_backup alter lastname  set not null;  
		
- SQL UNIQUE 约束  

	**UNIQUE 约束唯一标识数据库表中的每条记录**   
	**UNIQUE 和 PRIMARY KEY 约束均为列或列集合提供了唯一性的保证**   
	**PRIMARY KEY 拥有自动定义的 UNIQUE 约束**  
	**每个表可以有多个 UNIQUE 约束，但是每个表只能有一个 PRIMARY KEY 约束**   
	
		# SQL 在 "Persons" 表创建时在 "Id_P" 列创建 UNIQUE 约束
		create table test
		(
		Id_p int not null,
		lastname varchar(30) not null,
		unique (Id_p)
		)  
		
		# 命名 UNIQUE 约束，以及为多个列定义 UNIQUE 约束
		CREATE TABLE Persons
		(
		Id_P int NOT NULL,
		LastName varchar(255) NOT NULL,
		FirstName varchar(255),
		Address varchar(255),
		City varchar(255),
		CONSTRAINT uc_PersonID UNIQUE (Id_P,LastName)
		)

		# 当表已被创建时，如需在 "lastname" 列创建 UNIQUE 约束
		alter table test add unique (lastname);  
		# 如需命名 UNIQUE 约束，并定义多个列的 UNIQUE 约束  
		ALTER TABLE learn.persons ADD CONSTRAINT uc_personid UNIQUE (id_p, lastname);   
		# 如需撤销 UNIQUE 约束
		ALTER TABLE learn.persons DROP CONSTRAINT uc_personid;  
		
- SQL PRIMARY KEY 约束  

	**PRIMARY KEY 约束唯一标识数据库表中的每条记录**  
	**主键必须包含唯一的值**
	**主键列不能包含 NULL 值** 
	**每个表都应该有一个主键，并且每个表只能有一个主键** 
	**如果您使用 ALTER TABLE 语句添加主键，必须把主键列声明为不包含 NULL 值（在表首次创建时）**  
	

		# 下面的 SQL 在 "Persons" 表创建时在 "Id_P" 列创建 PRIMARY KEY 约束
		CREATE TABLE test
		(
		Id_P int NOT NULL,
		LastName varchar(255) NOT NULL,
		FirstName varchar(255),
		Address varchar(255),
		City varchar(255),
		PRIMARY KEY (Id_P)
		)  
		
		# 删除主键
		alter table test drop constraint test_pkey;  
		
		# 如果在表已存在的情况下为 "Id_P" 列创建 PRIMARY KEY 约束  
		ALTER TABLE learn.test ADD CONSTRAINT test_pkey PRIMARY KEY (id_p)   
		
- SQL FOREIGN KEY 约束

	**一个表中的 FOREIGN KEY 指向另一个表中的 PRIMARY KEY**  
	**FOREIGN KEY 约束用于预防破坏表之间连接的动作**
	**FOREIGN KEY 约束也能防止非法数据插入外键列，因为它必须是它指向的那个表中的值之一**  

		# 下面的 SQL 在 "Orders" 表创建时为 "Id_P" 列创建 FOREIGN KEY    		CREATE TABLE Orders
		(
		Id_O int NOT NULL,
		OrderNo int NOT NULL,
		Id_P int,
		PRIMARY KEY (Id_O),
		FOREIGN KEY (Id_P) REFERENCES Persons(Id_P)
		)  

		# 如果需要命名 FOREIGN KEY 约束，以及为多个列定义 FOREIGN KEY 约束，请使用下面的 SQL 语法
		CREATE TABLE Orders
		(
		Id_O int NOT NULL,
		OrderNo int NOT NULL,
		Id_P int,
		PRIMARY KEY (Id_O),
		CONSTRAINT fk_PerOrders FOREIGN KEY (Id_P)
		REFERENCES Persons(Id_P)
		)
		
		# 如果在 "Orders" 表已存在的情况下为 "Id_P" 列创建 FOREIGN KEY 约束，请使用下面的 SQL 
		ALTER TABLE learn.orders ADD CONSTRAINT orders_id_p_fkey FOREIGN KEY (id_p) REFERENCES persons(id_p);  
		
		# 如需撤销 FOREIGN KEY 约束，请使用下面的 SQL
		ALTER TABLE learn.orders DROP CONSTRAINT orders_id_p_fkey;  
		

- SQL CHECK 约束

	**CHECK 约束用于限制列中的值的范围**  
	**如果对单个列定义 CHECK 约束，那么该列只允许特定的值**  
	**如果对一个表定义 CHECK 约束，那么此约束会在特定的列中对值进行限制**  

		# 下面的 SQL 在 "Persons" 表创建时为 "Id_P" 列创建 CHECK 约束。CHECK 约束规定 "Id_P" 列必须只包含大于 0 的整数  
		CREATE TABLE Persons
		(
		Id_P int NOT NULL,
		LastName varchar(255) NOT NULL,
		FirstName varchar(255),
		Address varchar(255),
		City varchar(255),
		CHECK (Id_P>0)
		)  
	  
		# 如果需要命名 CHECK 约束，以及为多个列定义 CHECK 约束，请使用下面的 SQL 语法  
		CREATE TABLE Persons
		(
		Id_P int NOT NULL,
		LastName varchar(255) NOT NULL,
		FirstName varchar(255),
		Address varchar(255),
		City varchar(255),
		CONSTRAINT chk_Person CHECK (Id_P>0 AND City='Sandnes')
		)  
		
		# 如果在表已存在的情况下为 "Id_P" 列创建 CHECK 约束，请使用下面的 SQL 语法  
		ALTER TABLE learn.test ADD CONSTRAINT test_check CHECK (id_p > 0);  
		
		# 如果需要命名 CHECK 约束，以及为多个列定义 CHECK 约束，请使用下面的 SQL 语法 
		ALTER TABLE Persons ADD CONSTRAINT chk_Person CHECK (Id_P>0 AND City='Sandnes') ;
		
		# 如需撤销 CHECK 约束，请使用下面的 SQL
		alter table learn.test drop constraint chk_Person;  
		
- SQL DEFAULT 约束  

	**DEFAULT 约束用于向列中插入默认值**  
	**如果没有规定其他的值，那么会将默认值添加到所有的新记录**  
	
		# 下面的 SQL 在 "Persons" 表创建时为 "City" 列创建 DEFAULT 约束
		CREATE TABLE test
		(
		Id_P int NOT NULL,
		LastName varchar(255) NOT NULL,
		FirstName varchar(255),
		Address varchar(255),
		city varchar(255) default 'zhangye'
		)  
		
		# 如果在表已存在的情况下为 "City" 列创建 DEFAULT 约束，请使用下面的 SQL  
		ALTER TABLE Persons ALTER City SET DEFAULT 'SANDNES';  
		
		# 如需撤销 DEFAULT 约束，请使用下面的 SQL  
		ALTER TABLE Persons ALTER City DROP DEFAULT;  
		
**21、SQL CREATE INDEX 语句**  

**CREATE INDEX 语句用于在表中创建索引**
**在不读取整个表的情况下，索引使数据库应用程序可以更快地查找数据**  

**在表中创建索引，以便更加快速高效地查询数据，用户无法看到索引，它们只能被用来加速搜索/查询**    
**注意：更新一个包含索引的表需要比更新一个没有索引的表更多的时间，这是由于索引本身也需要更新。因此，理想的做法是仅仅在常常被搜索的列（以及表）上面创建索引**  

在表上创建一个简单的索引。允许使用重复的值  

	CREATE INDEX index_name
	ON table_name (column_name)  
	
在表上创建一个唯一的索引。唯一的索引意味着两个行不能拥有相同的索引值  

	CREATE UNIQUE INDEX index_name
	ON table_name (column_name)  
	
CREATE INDEX 实例  

	# 创建一个简单的索引，名为 "PersonIndex"，在 Person 表的 LastName 列
	CREATE INDEX PersonIndex
	ON Person (LastName)   
	
	# 希望以降序索引某个列中的值，可以在列名称之后添加保留字 DESC
	CREATE INDEX PersonIndex
	ON Person (LastName DESC)   
	
	# 希望索引不止一个列，可以在括号中列出这些列的名称，用逗号隔开
	CREATE INDEX PersonIndex
	ON Person (LastName, FirstName)  
	
**22、SQL 撤销索引、表以及数据库**  

**通过使用 DROP 语句，可以轻松地删除索引、表和数据库**  

	# 使用 DROP INDEX 命令删除表格中的索引
	DROP INDEX index_name  
	
	# DROP TABLE 语句用于删除表（表的结构、属性以及索引也会被删除）
	DROP TABLE 表名称
	
	# DROP DATABASE 语句用于删除数据库  
	DROP DATABASE 数据库名称

**SQL TRUNCATE TABLE 语句**  

	# RUNCATE TABLE 命令（仅仅删除表格中的数据）
	TRUNCATE TABLE 表名称

**23、SQL ALTER TABLE 语句**  

**ALTER TABLE 语句用于在已有的表中添加、修改或删除列**  

	# 在表中添加列，请使用下列语法
	ALTER TABLE table_name ADD column_name datatype  
	alter table test add birthday varchar(255);  
	
	# 改变表中列的数据类型，请使用下列语法  
	ALTER TABLE table_name ALTER column_name type datatype  
	alter table test alter birthday type char(30);
	
	# 删除表中的列，请使用下列语法  
	ALTER TABLE table_name DROP COLUMN column_name
	alter table test drop column birthday;  
	
**24、SQL AUTO INCREMENT 字段**  

**postgres中通过触发器实现自增列**  

	# 用于 MySQL 的语法
	CREATE TABLE Persons
	(
	P_Id int NOT NULL AUTO_INCREMENT,
	LastName varchar(255) NOT NULL,
	FirstName varchar(255),
	Address varchar(255),
	City varchar(255),
	PRIMARY KEY (P_Id)
	)
	
	ALTER TABLE Persons AUTO_INCREMENT=100  

**25、视图是可视化的表**  

**在 SQL 中，视图是基于 SQL 语句的结果集的可视化的表**  
**视图包含行和列，就像一个真实的表。视图中的字段就是来自一个或多个数据库中的真实的表中的字段。我们可以向视图添加 SQL 函数、WHERE 以及 JOIN 语句，我们也可以提交数据，就像这些来自于某个单一的表**  
**数据库的设计和结构不会受到视图中的函数、where 或 join 语句的影响**  
**可以从某个查询内部、某个存储过程内部，或者从另一个视图内部来使用视图。通过向视图添加函数、join 等等，我们可以向用户精确地提交我们希望提交的数据**


	CREATE VIEW view_name AS
	SELECT column_name(s)
	FROM table_name
	WHERE condition  

**视图总是显示最近的数据。每当用户查询视图时，数据库引擎通过使用 SQL 语句来重建数据**  

	# 查看视图
	SELECT * FROM view_name
	
	# 可以使用下面的语法来更新视图(postgresql采用DROP VIEW再CREATE VIEW的形式)   
	DROP VIEW view_name  

	CREATE VIEW view_name AS
	SELECT column_name(s)
	FROM table_name
	WHERE condition    
	
**26、SQL Date 函数**  

**Postgresql时间函数**  

函数|	返回类型|	描述|示例|结果
---|---|----|----|-----|
age(timestamp, timestamp)	|interval	|计算两个时间戳的时间间隔|select age(timestamp '2001-04-10',timestamp '1957-06-13');|43 years 9 mons 27 days
age(timestamp)	|interval	|计算current_date与入参时间戳的时间间隔|select age(timestamp '2016-07-07 12:00:00');|12:00:00
clock_timestamp()|	timestamp with time zone|当前时间戳（语句执行时变化）|	select clock_timestamp();	|2016-07-08 15:14:04.197732-07
current_date|	date|	当前日期	|select current_date;|2016-07-08
current_time	|time with time zone	|当前时间|select current_time;|15:15:56.394651-07
current_timestamp	|timestamp with time zone|当前时间戳|select current_timestamp;|2016-07-08 15:16:50.485864-07  
date_part(text, timestamp)| double precision|获取时间戳中的某个子域，其中text可以为year,month,day,hour,minute,second等	|select date_part('year',timestamp'2016-07-08 12:05:06'),date_part('month',timestamp'2016-07-08 12:05:06'),date_part('day',timestamp'2016-07-08 12:05:06'),date_part('hour',timestamp'2016-07-08 12:05:06'),date_part('minute',timestamp'2016-07-08 12:05:06'),date_part('second',timestamp'2016-07-08 12:05:06');|2016 \| 7 \| 8 \| 12 \| 5 \| 6
date_part(text, interval)	|double precision|功能同上，只是第二个入参为时间间隔|select date_part('hour',interval'1 day 13:00:12');|13
date_trunc(text, timestamp)|timestamp	|将时间戳截断成指定的精度，指定精度后面的子域用0补充|select date_trunc('hour',timestamp'2016-07-08 22:30:33');|2016-07-08 22:00:00
date_trunc(text, interval)	|interval	|功能同上，只是第二个入参为时间间隔|select date_trunc('hour',interval'1 year 2 mon 3 day 22:30:33');|1 year 2 mons 3 days 22:00:00
extract(field from timestamp)	|double precision|功能同date_part(text, timestamp)|select extract(hour from timestamp'2016-07-08 22:30:29');|22
extract(field from interval)|	double precision|功能同date_part(text, interval)|select extract(hour from interval'1 day 13:00:12');|13
isfinite(date)|	boolean	|测试是否为有穷日期|select isfinite(date'2016-07-08'),isfinite(date'infinity');|t,f
isfinite(timestamp)|	boolean	|测试是否为有穷时间戳|select isfinite(timestamp'2016-07-08');|t
isfinite(interval)|	boolean|	测试是否为有穷时间间隔|select isfinite(interval'1day 23:02:12');|t
justify_days(interval)|interval	|按照每月30天调整时间间隔|select justify_days(interval'1year 45days 23:00:00');|	1 year 1 mon 15 days 23:00:00
justify_hours(interval)	|interval	|按照每天24小时调整时间间隔|select justify_hours(interval'1year 45days 343hour');	|1 year 59 days 07:00:00
justify_interval(interval)	|interval|同时使用justify_days(interval)和justify_hours(interval)|select justify_interval(interval'1year 45days 343hour');|1 year 1 mon 29 days 07:00:00
localtime	|time	|当日时间	|select localtime;|15:45:18.892224
localtimestamp|	timestamp|	当日日期和时间|select localtimestamp;|2016-07-08 15:46:55.181583
make_interval(years int DEFAULT 0, months int DEFAULT 0, weeks int DEFAULT 0, days int DEFAULT 0, hours int DEFAULT 0, mins int DEFAULT 0,secs double precision DEFAULT 0.0)| interval|	创建一个时间间隔	|select make_interval(1,hours=>3);|1 year 03:00:00
make_time(hour int, min int, sec double precision)|time|创建一个时间|select make_time(9,21,23);|	09:21:23  
make_timestamp(year int, month int, day int, hour int, min int,sec double precision)|timestamp	|创建一个时间戳|	select make_timestamp(2016,7,8,22,55,23.5);|2016-07-08 22:55:23.5  
make_timestamptz(year int, month int, day int, hour int, min int, sec double precision, [ timezone text ])	|timestamp with time zone|创建一个带有时区的时间戳|select make_timestamptz(2016,7,8,22,55,23.5);|2016-07-08 22:55:23.5-07  
now()|timestamp with time zone	|当前日期和时间|select now();|2016-07-08 15:55:30.873537-07
statement_timestamp()	|timestamp with time zone |同now()|select statement_timestamp();|2016-07-08 15:56:07.259956-07
timeofday()|text	|当前日期和时间，包含周几，功能与clock_timestamp()类似|select timeofday();|Fri Jul 08 15:57:51.277239 2016 PDT
transaction_timestamp()	|timestamp with time zone|事务开始时的时间戳|select transaction_timestamp();|2016-07-08 16:01:25.007153-07
to_timestamp(double precision)|	timestamp with time zone|Convert Unix epoch(seconds since 1970-01-0100:00:00+00) to timestamp|select to_timestamp(1284352323);|2010-09-12 21:32:03-07
pg_sleep(seconds double precision);|	 |当前会话休眠seconds秒|select pg_sleep(5);

pg_sleep_for(interval)|	 |	当前会话休眠多长时间的间隔|select pg_sleep_for('5 seconds');

pg_sleep_until(timestamp with time zone)| |当前会话休眠至什么时间点|select pg_sleep_until('2016-07-08 23:59:59');	 

**27、SQL NULL 值**  

**NULL 值是遗漏的未知数据。默认地，表的列可以存放 NULL 值**  
**如果表中的某个列是可选的，那么我们可以在不向该列添加值的情况下插入新记录或更新已有的记录。这意味着该字段将以 NULL 值保存。NULL 值的处理方式与其他值不同。NULL 用作未知的或不适用的值的占位符。**  

**无法比较 NULL 和 0；它们是不等价的,无法使用比较运算符来测试 NULL 值，比如 =, <, 或者 <>,我们必须使用 IS NULL 和 IS NOT NULL 操作符**   

SQL IS NULL  

	SELECT LastName,FirstName,Address FROM Persons WHERE Address IS NULL  
	
SQL IS NOT NULL

	SELECT LastName,FirstName,Address FROM Persons WHERE Address IS NOT NULL  
	



			















	  




		





























 



















  




