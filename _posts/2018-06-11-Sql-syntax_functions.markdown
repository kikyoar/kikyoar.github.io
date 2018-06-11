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

















 



















  




