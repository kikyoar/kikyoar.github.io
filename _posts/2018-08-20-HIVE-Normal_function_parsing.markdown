---
layout:     post
title:      "Hive常用函数大全"
subtitle:   "关系、数学、逻辑、字符串、数值、日期、条件、集合统计、复杂类型"
date:       2018-08-20
author:     "kikyoar"
header-img: "img/post-bg-hadoop-version.jpg"
tags:
    - Hadoop
---  



## 字符串函数

**字符串长度计算函数：length**  

语法: length(string A)  
返回值: int  
说明：返回字符串A的长度  
例子：  

	hive> select length('iteblog') from iteblog;
	7    

**字符串反转函数：reverse**    

语法: reverse(string A)  
返回值: string  
说明：返回字符串A的反转结果  
例子：    

	hive> select reverse(iteblog') from iteblog;
	golbeti  
	
**字符串连接函数：concat**  

语法: concat(string A, string B…)  
返回值: string  
说明：返回输入字符串连接后的结果，支持任意个输入字符串  
例子：  

	hive> select concat('www','.iteblog','.com') from iteblog;
	www.iteblog.com  
	
**带分隔符字符串连接函数：concat_ws**

语法: concat_ws(string SEP, string A, string B…)  
返回值: string  
说明：返回输入字符串连接后的结果，SEP表示各个字符串间的分隔符  
例子：  

	hive> select concat_ws('.','www','iteblog','com') from iteblog;
	www.iteblog.com  
	
	
**字符串截取函数：substr,substring**

语法: substr(string A, int start),substring(string A, int start)  
返回值: string  
说明：返回字符串A从start位置到结尾的字符串  
例子：  

	hive> select substr('iteblog',3) from iteblog;
	eblog
	hive>  selectsubstr('iteblog',-1) from iteblog; 
	g  
	
**字符串截取函数：substr,substring**  

语法: substr(string A, int start, int len),substring(string A, intstart, int len)  
返回值: string  
说明：返回字符串A从start位置开始，长度为len的字符串  
例子：  

	hive> select substr('abcde',3,2) from iteblog;
	cd
	hive> select substring('abcde',3,2) from iteblog;
	cd
	hive>select substring('abcde',-2,2) from iteblog;
	de
	
**字符串转大写函数：upper,ucase**

语法: upper(string A) ucase(string A)  
返回值: string  
说明：返回字符串A的大写格式  
例子：    

	hive> select upper('abSEd') from iteblog;
	ABSED
	hive> select ucase('abSEd') from iteblog;
	ABSED

**字符串转小写函数：lower,lcase**

语法: lower(string A) lcase(string A)  
返回值: string  
说明：返回字符串A的小写格式  
例子：  

	hive> select lower('abSEd') from iteblog;
	absed
	hive> select lcase('abSEd') from iteblog;
	absed  
	
**去空格函数：trim**

语法: trim(string A)  
返回值: string  
说明：去除字符串两边的空格  
例子：  

	hive> select trim(' abc ') from iteblog;
	abc  
	
**左边去空格函数：ltrim**  

语法: ltrim(string A)  
返回值: string  
说明：去除字符串左边的空格  
例子：  

	hive> select ltrim(' abc ') from iteblog;
	abc  
	
**右边去空格函数：rtrim**  

语法: rtrim(string A)  
返回值: string  
说明：去除字符串右边的空格  
例子：  

	hive> select rtrim(' abc ') from iteblog;
	abc  
	
**正则表达式替换函数：regexp_replace**

语法: regexp\_replace(string A, string B, string C)
返回值: string  
说明：将字符串A中的符合java正则表达式B的部分替换为C。注意，在有些情况下要使用转义字符,类似oracle中的regexp_replace函数  
例子：  

	hive> select regexp_replace('foobar', 'oo|ar', '') from iteblog;
	fb  

**正则表达式解析函数：regexp_extract**

语法: regexp_extract(string subject, string pattern, int index)  
返回值: string  
说明：将字符串subject按照pattern正则表达式的规则拆分，返回index指定的字符  

第一参数：要处理的字段
第二参数：需要匹配的正则表达式
第三个参数：
		0 是显示与之匹配的整个字符串
		1 是显示第一个括号里面的
		2 是显示第二个括号里面的字段...  
例子：

	select
	regexp_extract('x=a3&x=18abc&x=2&y=3&x=4','x=([0-9]+)([a-z]+)',0),  -- x=18abc
	regexp_extract('x=a3&x=18abc&x=2&y=3&x=4','^x=([a-z]+)([0-9]+)',0), -- x=a3
	 
	regexp_extract('https://detail.tmall.com/item.htm?spm=608.7065813.ne.1.Ni3rsN&id=522228774076&tracelog=fromnonactive','id=([0-9]+)',0),    -- id=522228774076
	regexp_extract('https://detail.tmall.com/item.htm?spm=608.7065813.ne.1.Ni3rsN&id=522228774076&tracelog=fromnonactive','id=([0-9]+)',1),    -- 522228774076
	 
	regexp_extract('http://a.m.taobao.com/i41915173660.htm','i([0-9]+)',0),            -- i41915173660
	regexp_extract('http://a.m.taobao.com/i41915173660.htm','i([0-9]+)',1)             -- 41915173660
	 
	from test.dual;
	
注意，在有些情况下要使用转义字符，下面的等号要用双竖线转义，这是java正则表达式的规则。

		 select data_field,
	     regexp_extract(data_field,'.*?bgStart\\=([^&]+)',1) as aaa,
	     regexp_extract(data_field,'.*?contentLoaded_headStart\\=([^&]+)',1) as bbb,
	     regexp_extract(data_field,'.*?AppLoad2Req\\=([^&]+)',1) as ccc
	     from pt_nginx_loginlog_st
	     where pt = '2012-03-26'limit 2;  
	     
复杂例子：

	select substring(substr(regexp_extract(uri,'lon=([0-9]+)\.([0-9]+)',0),5),0,9) lon,  
	substring(substr(regexp_extract(uri,'lat=([0-9]+)\.([0-9]+)',0),5),0,9) lat  
	from o_lte_s1u_http where dh='2018081920' LIMIT 100;  
	
**URL解析函数：parse_url**

语法: parse_url(string urlString, string partToExtract [, stringkeyToExtract])  
返回值: string  
说明：返回URL中指定的部分。partToExtract的有效值为：HOST, PATH, QUERY, REF, PROTOCOL, AUTHORITY, FILE, and USERINFO  
例子：  

	hive> select parse_url('http://iteblog.com?weixin=iteblog_hadoop', 'HOST') from iteblog;
	iteblog.com
	hive> select parse_url('http://iteblog.com?weixin=iteblog_hadoop','QUERY','weixin') from iteblog;
	iteblog_hadoop  
	
**json解析函数：get\_json\_object**

语法: get\_json\_object(string json\_string, string path)  
返回值: string  
说明：解析json的字符串json\_string,返回path指定的内容。如果输入的json字符串无效，那么返回NULL  
例子：  

	hive> select get_json_object('{"store":
		>  {"fruit":\[{"weight":8,"type":"apple"},{"weight":9,"type":"pear"}],
		>   "bicycle":{"price":19.95,"color":"red"}
		>   },
		> "email":"amy@only_for_json_udf_test.net",
		>  "owner":"amy"
		> }
		> ','$.owner') from iteblog;
		
		amy


**空格字符串函数：space**

语法: space(int n)  
返回值: string  
说明：返回长度为n的字符串  
例子：

	hive> select space(10) from iteblog;
	hive> select length(space(10)) from iteblog;
	10


**重复字符串函数：repeat**  

语法: repeat(string str, int n)  
返回值: string  
说明：返回重复n次后的str字符串  
例子：  

	hive> select repeat('abc',5) from iteblog;
	abcabcabcabcabc  

**首字符ascii函数：ascii**  

语法: ascii(string str)  
返回值: int  
说明：返回字符串str第一个字符的ascii码  
例子：  

	hive> select ascii('abcde') from iteblog;
	97  
	
**左补足函数：lpad**  

语法: lpad(string str, int len, string pad)  
返回值: string  
说明：将str进行用pad进行左补足到len位  
例子：  

	hive> select lpad('abc',10,'td') from iteblog;
	tdtdtdtabc
注意：与GP，ORACLE不同，pad 不能默认

**右补足函数：rpad**

语法: rpad(string str, int len, string pad)  
返回值: string  
说明：将str进行用pad进行右补足到len位  
例子：  

	hive> select rpad('abc',10,'td') from iteblog;
	abctdtdtdt  
	
**分割字符串函数: split**  

语法: split(string str, stringpat)  
返回值: array  
说明: 按照pat字符串分割str，会返回分割后的字符串数组  
例子：  

	hive> select split('abtcdtef','t') from iteblog;
	["ab","cd","ef"]

**集合查找函数:find\_in\_set**  

语法: find\_in\_set(string str, string strList)  
返回值: int  
说明: 返回str在strlist第一次出现的位置，strlist是用逗号分割的字符串。如果没有找该str字符，则返回0  
例子：  

	hive> select find_in_set('ab','ef,ab,de') from iteblog;
	2
	hive> select find_in_set('at','ef,ab,de') from iteblog;
	0
	


## 关系运算


**等值比较: =**  

语法：A=B  
操作类型：所有基本类型  
描述: 如果表达式A与表达式B相等，则为TRUE；否则为FALSE  

	hive> select 1 from iteblog where 1=1;
	1

**不等值比较: <>**  

语法: A <> B  
操作类型: 所有基本类型  
描述: 如果表达式A为NULL，或者表达式B为NULL，返回NULL；如果表达式A与表达式B不相等，则为TRUE；否则为FALSE  

	hive> select 1 from iteblog where 1 <> 2;
	1
	
**小于比较: <**  

语法: A < B  
操作类型：所有基本类型  
描述: 如果表达式A为NULL，或者表达式B为NULL，返回NULL；如果表达式A小于表达式B，则为TRUE；否则为FALSE  

	hive> select 1 from iteblog where 1 < 2;
	1
	
**小于等于比较: <=**  

语法: A <= B  
操作类型: 所有基本类型  
描述: 如果表达式A为NULL，或者表达式B为NULL，返回NULL；如果表达式A小于或者等于表达式B，则为TRUE；否则为FALSE  

	hive> select 1 from iteblog where 1 < = 1;
	1  
	

**大于比较: >**  

语法: A > B  
操作类型: 所有基本类型  
描述: 如果表达式A为NULL，或者表达式B为NULL，返回NULL；如果表达式A大于表达式B，则为TRUE；否则为FALSE  

	hive> select 1 from iteblog where 2 > 1;
	1  
	
**大于等于比较: >=**  

语法: A >= B  
操作类型: 所有基本类型  
描述: 如果表达式A为NULL，或者表达式B为NULL，返回NULL；如果表达式A大于或者等于表达式B，则为TRUE；否则为FALSE  

	hive> select 1 from iteblog where 1 >= 1;
	1  
	
注意：String的比较要注意(常用的时间比较可以先 to_date 之后再比较)

	hive> select * from iteblog;
	OK
	2011111209 00:00:00     2011111209
	 
	hive> select a, b, a<b, a>b, a=b from iteblog;
	2011111209 00:00:00     2011111209      false   true    false

**空值判断: IS NULL**  

语法: A IS NULL  
操作类型: 所有类型  
描述: 如果表达式A的值为NULL，则为TRUE；否则为FALSE  

	hive> select 1 from iteblog where null is null;
	1

**非空判断: IS NOT NULL**  

语法: A IS NOT NULL  
操作类型: 所有类型  
描述: 如果表达式A的值为NULL，则为FALSE；否则为TRUE  

	hive> select 1 from iteblog where 1 is not null;
	1  

**LIKE比较: LIKE**  

语法: A LIKE B  
操作类型: strings  
描述: 如果字符串A或者字符串B为NULL，则返回NULL；如果字符串A符合表达式B 的正则语法，则为TRUE；否则为FALSE。B中字符”_”表示任意单个字符，而字符”%”表示任意数量的字符  

	hive> select 1 from iteblog where 'football' like 'foot%';
	1
	hive> select 1 from iteblog where 'football' like 'foot____';
	1
注意：否定比较时候用NOT A LIKE B

	hive> select 1 from iteblog where NOT 'football' like 'fff%';
	1  

**JAVA的LIKE操作: RLIKE**  

语法: A RLIKE B  
操作类型: strings  
描述: 如果字符串A或者字符串B为NULL，则返回NULL；如果字符串A符合JAVA正则表达式B的正则语法，则为TRUE；否则为FALSE  

	hive> select 1 from iteblog where 'footbar’ rlike '^f.*r$’;
	1

注意：判断一个字符串是否全为数字：

	hive>select 1 from iteblog where '123456' rlike '^\\d+$';
	1
	hive> select 1 from iteblog where '123456aa' rlike '^\\d+$';  
	
**REGEXP操作: REGEXP**  

语法: A REGEXP B  
操作类型: strings  
描述: 功能与RLIKE相同  

	hive> select 1 from iteblog where 'footbar' REGEXP '^f.*r$';
	1  


## 数学运算  

**加法操作: +**  

语法: A + B  
操作类型：所有数值类型  
说明：返回A与B相加的结果。结果的数值类型等于A的类型和B的类型的最小父类型  
比如，int + int 一般结果为int类型，而 int + double 一般结果为double类型  

	hive> select 1 + 9 from iteblog;
	10
	hive> create table iteblog as select 1 + 1.2 from iteblog;
	hive> describe iteblog;
	_c0     double  


**减法操作: -**  

语法: A – B  
操作类型：所有数值类型  
说明：返回A与B相减的结果。结果的数值类型等于A的类型和B的类型的最小父类型  
比如，int – int 一般结果为int类型，而 int – double 一般结果为double类型  

	hive> select 10 – 5 from iteblog;
	5
	hive> create table iteblog as select 5.6 – 4 from iteblog;
	hive> describe iteblog;
	_c0     double  

**乘法操作: \***  

语法: A * B  
操作类型：所有数值类型  
说明：返回A与B相乘的结果。结果的数值类型等于A的类型和B的类型的最小父类型    
注意，如果A乘以B的结果超过默认结果类型的数值范围，则需要通过cast将结果转换成范围更大的数值类型

	hive> select 40 * 5 from iteblog;
	200  

**除法操作: /**  

语法: A / B  
操作类型：所有数值类型  
说明：返回A除以B的结果。结果的数值类型为double  

	hive> select 40 / 5 from iteblog;
	8.0  
	
注意：hive中最高精度的数据类型是double,只精确到小数点后16位，在做除法运算的时候要特别注意

	hive>select ceil(28.0/6.999999999999999999999) from iteblog limit 1;    
	结果为4
	hive>select ceil(28.0/6.99999999999999) from iteblog limit 1;           
	结果为5  
	
**取余操作: %**  

语法: A % B  
操作类型：所有数值类型  
说明：返回A除以B的余数。结果的数值类型等于A的类型和B的类型的最小父类型   

	hive> select 41 % 5 from iteblog;
	1
	hive> select 8.4 % 4 from iteblog;
	0.40000000000000036  
	
注意：精度在hive中是个很大的问题，类似这样的操作最好通过round指定精度  

	hive> select round(8.4 % 4 , 2) from iteblog;
	0.4  
	
**位与操作: &**  

语法: A & B  
操作类型：所有数值类型  
说明：返回A和B按位进行与操作的结果。结果的数值类型等于A的类型和B的类型的最小父类型    

	hive> select 4 & 8 from iteblog;
	0
	hive> select 6 & 4 from iteblog;
	4  
	
**位或操作: |**  

语法: A | B  
操作类型：所有数值类型    
说明：返回A和B按位进行或操作的结果。结果的数值类型等于A的类型和B的类型的最小父类型     

	hive> select 4 | 8 from iteblog;
	12
	hive> select 6 | 8 from iteblog;
	14  
	
**位异或操作: ^**  

语法: A ^ B  
操作类型：所有数值类型    
说明：返回A和B按位进行异或操作的结果。结果的数值类型等于A的类型和B的类型的最小父类型   
 
	hive> select 4 ^ 8 from iteblog;
	12
	hive> select 6 ^ 4 from iteblog;
	2  
	
**位取反操作: ~**  

语法: ~A  
操作类型：所有数值类型  
说明：返回A按位取反操作的结果。结果的数值类型等于A的类型  

	hive> select ~6 from iteblog;
	-7
	hive> select ~4 from iteblog;
	-5  
	
## 逻辑运算  

**逻辑与操作: AND**  

语法: A AND B  
操作类型：boolean  
说明：如果A和B均为TRUE，则为TRUE；否则为FALSE。如果A为NULL或B为NULL，则为NULL  

	hive> select 1 from iteblog where 1=1 and 2=2;
	1  
	
**逻辑或操作: OR**  

语法: A OR B  
操作类型：boolean  
说明：如果A为TRUE，或者B为TRUE，或者A和B均为TRUE，则为TRUE；否则为FALSE  

	hive> select 1 from iteblog where 1=2 or 2=2;
	1  

**逻辑非操作: NOT**  

语法: NOT A  
操作类型：boolean  
说明：如果A为FALSE，或者A为NULL，则为TRUE；否则为FALSE  

	hive> select 1 from iteblog where not 1=2;
	1  

## 数值计算  

**取整函数: round**  

语法: round(double a)  
返回值: BIGINT  
说明: 返回double类型的整数值部分 （遵循四舍五入）  

	hive> select round(3.1415926) from iteblog;
	3
	hive> select round(3.5) from iteblog;
	4
	hive> create table iteblog as select round(9542.158) from iteblog;
	hive> describe iteblog;
	_c0     bigint  
	
**指定精度取整函数: round**  

语法: round(double a, int d)  
返回值: DOUBLE  
说明: 返回指定精度d的double类型  

	hive> select round(3.1415926,4) from iteblog;
	3.1416  
	
**向下取整函数: floor**  

语法: floor(double a)  
返回值: BIGINT  
说明: 返回等于或者小于该double变量的最大的整数  

	hive> select floor(3.1415926) from iteblog;
	3
	hive> select floor(25) from iteblog;
	25  

**向上取整函数: ceil**  

语法: ceil(double a)  
返回值: BIGINT  
说明: 返回等于或者大于该double变量的最小的整数  

	hive> select ceil(3.1415926) from iteblog;
	4
	hive> select ceil(46) from iteblog;
	46  
	
**向上取整函数: ceiling**  

语法: ceiling(double a)  
返回值: BIGINT  
说明: 与ceil功能相同  

	hive> select ceiling(3.1415926) from iteblog;
	4
	hive> select ceiling(46) from iteblog;
	46  

**取随机数函数: rand**  

语法: rand(),rand(int seed)  
返回值: double  
说明: 返回一个0到1范围内的随机数。如果指定种子seed，则会等到一个稳定的随机数序列  

	hive> select rand() from iteblog;
	0.5577432776034763
	hive> select rand() from iteblog;
	0.6638336467363424
	hive> select rand(100) from iteblog;
	0.7220096548596434
	hive> select rand(100) from iteblog;
	0.7220096548596434  
	
**自然指数函数: exp**  

语法: exp(double a)  
返回值: double  
说明: 返回自然对数e的a次方  

	hive> select exp(2) from iteblog;
	7.38905609893065  
	
**自然对数函数: ln**  

语法: ln(double a)  
返回值: double  
说明: 返回a的自然对数  

	hive> select ln(7.38905609893065) from iteblog;
	2.0  
	
**以10为底对数函数: log10**  

语法: log10(double a)  
返回值: double  
说明: 返回以10为底的a的对数  

	hive> select log10(100) from iteblog;
	2.0  
	
**以2为底对数函数: log2**  

语法: log2(double a)  
返回值: double  
说明: 返回以2为底的a的对数  

	hive> select log2(8) from iteblog;
	3.0  

**对数函数: log**  

语法: log(double base, double a)  
返回值: double  
说明: 返回以base为底的a的对数  

	hive> select log(4,256) from iteblog;
	4.0  

**幂运算函数: pow、power**  

语法: pow(double a, double p)类同于power(double a, double p)    
返回值: double  
说明: 返回a的p次幂  

	hive> select pow(2,4) from iteblog;
	16.0  


	hive> select power(2,4) from iteblog;
	16.0  


**开平方函数: sqrt**  

语法: sqrt(double a)  
返回值: double  
说明: 返回a的平方根  

	hive> select sqrt(16) from iteblog;
	4.0  
	
**二进制函数: bin**  

语法: bin(BIGINT a)  
返回值: string  
说明: 返回a的二进制代码表示  

	hive> select bin(7) from iteblog;
	111  
	
**十六进制函数: hex**  

语法: hex(BIGINT a)  
返回值: string  
说明: 如果变量是int类型，那么返回a的十六进制表示；如果变量是string类型，则返回该字符串的十六进制表示  

	hive> select hex(17) from iteblog;
	11
	hive> select hex(‘abc’) from iteblog;
	616263  

**反转十六进制函数: unhex**  

语法: unhex(string a)  
返回值: string  
说明: 返回该十六进制字符串所代码的字符串  

	hive> select unhex(‘616263’) from iteblog;
	abc
	hive> select unhex(‘11’) from iteblog;
	-
	hive> select unhex(616263) from iteblog;
	abc  

**进制转换函数: conv**  

语法: conv(BIGINT num, int from_base, int to_base)  
返回值: string  
说明: 将数值num从from_base进制转化到to_base进制  

	hive> select conv(17,10,16) from iteblog;
	11
	hive> select conv(17,10,2) from iteblog;
	10001  

**绝对值函数: abs**  

语法: abs(double a) abs(int a)  
返回值: double int  
说明: 返回数值a的绝对值  

	hive> select abs(-3.9) from iteblog;
	3.9
	hive> select abs(10.9) from iteblog;
	10.9  

**正取余函数: pmod**  

语法: pmod(int a, int b),pmod(double a, double b)  
返回值: int double  
说明: 返回正的a除以b的余数  

	hive> select pmod(9,4) from iteblog;
	1
	hive> select pmod(-9,4) from iteblog;
	3  

**正弦函数: sin**  

语法: sin(double a)  
返回值: double  
说明: 返回a的正弦值   
	
	hive> select sin(0.8) from iteblog;
	0.7173560908995228  

**反正弦函数: asin**  

语法: asin(double a)  
返回值: double  
说明: 返回a的反正弦值  

	hive> select asin(0.7173560908995228) from iteblog;
	0.8  

**余弦函数: cos**  

语法: cos(double a)  
返回值: double  
说明: 返回a的余弦值  

	hive> select cos(0.9) from iteblog;
	0.6216099682706644  

**反余弦函数: acos**  

语法: acos(double a)  
返回值: double  
说明: 返回a的反余弦值  

	hive> select acos(0.6216099682706644) from iteblog;
	0.9  

**positive函数: positive**  

语法: positive(int a), positive(double a)  
返回值: int double  
说明: 返回a  

	hive> select positive(-10) from iteblog;
	-10
	hive> select positive(12) from iteblog;
	12  

**negative函数: negative**  

语法: negative(int a), negative(double a)    
返回值: int double  
说明: 返回-a  

	hive> select negative(-5) from iteblog;
	5
	hive> select negative(8) from iteblog;
	-8  

## 日期函数  

**UNIX时间戳转日期函数: from_unixtime**  

语法: from_unixtime(bigint unixtime[, string format])  
返回值: string  
说明: 转化UNIX时间戳（从1970-01-01 00:00:00 UTC到指定时间的秒数）到当前时区的时间格式  

	hive> select from_unixtime(1323308943,'yyyyMMdd') from iteblog;
	20111208  
	
**获取当前UNIX时间戳函数: unix_timestamp**  

语法: unix_timestamp()  
返回值: bigint  
说明: 获得当前时区的UNIX时间戳  

	hive> select unix_timestamp() from iteblog;
	1323309615  
	
**日期转UNIX时间戳函数: unix_timestamp**  

语法: unix_timestamp(string date)  
返回值: bigint  
说明: 转换格式为"yyyy-MM-dd HH:mm:ss"的日期到UNIX时间戳。如果转化失败，则返回0  

	hive> select unix_timestamp('2011-12-07 13:01:03') from iteblog;
	1323234063  
	
**指定格式日期转UNIX时间戳函数: unix_timestamp**  

语法: unix_timestamp(string date, string pattern)  
返回值: bigint  
说明: 转换pattern格式的日期到UNIX时间戳。如果转化失败，则返回0  

	hive> select unix_timestamp('20111207 13:01:03','yyyyMMdd HH:mm:ss') from iteblog;
	1323234063  
	
**日期时间转日期函数: to_date**  

语法: to_date(string timestamp)  
返回值: string  
说明: 返回日期时间字段中的日期部分  

	hive> select to_date('2011-12-08 10:03:01') from iteblog;
	2011-12-08  
	
**日期转年函数: year**  

语法: year(string date)  
返回值: int  
说明: 返回日期中的年  
	
	hive> select year('2011-12-08 10:03:01') from iteblog;
	2011
	hive> select year('2012-12-08') from iteblog;
	2012  

**日期转月函数: month**  

语法: month (string date)  
返回值: int  
说明: 返回日期中的月份  

	hive> select month('2011-12-08 10:03:01') from iteblog;
	12
	hive> select month('2011-08-08') from iteblog;
	8  

**日期转天函数: day**  

语法: day (string date)  
返回值: int  
说明: 返回日期中的天  

	hive> select day('2011-12-08 10:03:01') from iteblog;
	8
	hive> select day('2011-12-24') from iteblog;
	24  

**日期转小时函数: hour**  

语法: hour (string date)  
返回值: int  
说明: 返回日期中的小时  

	hive> select hour('2011-12-08 10:03:01') from iteblog;
	10  

**日期转分钟函数: minute**  

语法: minute (string date)  
返回值: int  
说明: 返回日期中的分钟  

	hive> select minute('2011-12-08 10:03:01') from iteblog;
	3  

**日期转秒函数: second**  

语法: second (string date)  
返回值: int  
说明: 返回日期中的秒  

	hive> select second('2011-12-08 10:03:01') from iteblog;
	1  

**日期转周函数: weekofyear**  

语法: weekofyear (string date)  
返回值: int  
说明: 返回日期在当前的周数  

	hive> select weekofyear('2011-12-08 10:03:01') from iteblog;
	49  

**日期比较函数: datediff**  

语法: datediff(string enddate, string startdate)  
返回值: int  
说明: 返回结束日期减去开始日期的天数  

	hive> select datediff('2012-12-08','2012-05-09') from iteblog;
	213  

**日期增加函数: date_add**  

语法: date_add(string startdate, int days)  
返回值: string  
说明: 返回开始日期startdate增加days天后的日期  

	hive> select date_add('2012-12-08',10) from iteblog;
	2012-12-18  

**日期减少函数: date_sub**  

语法: date_sub (string startdate, int days)  
返回值: string  
说明: 返回开始日期startdate减少days天后的日期  

	hive> select date_sub('2012-12-08',10) from iteblog;
	2012-11-28  

## 条件函数  

**If函数: if**  

语法: if(boolean testCondition, T valueTrue, T valueFalseOrNull)  
返回值: T  
说明: 当条件testCondition为TRUE时，返回valueTrue；否则返回valueFalseOrNull  

	hive> select if(1=2,100,200) from iteblog;
	200
	hive> select if(1=1,100,200) from iteblog;
	100  

**非空查找函数: COALESCE**  

语法: COALESCE(T v1, T v2, …)  
返回值: T  
说明: 返回参数中的第一个非空值；如果所有值都为NULL，那么返回NULL  

	hive> select COALESCE(null,'100','50′) from iteblog;
	100  
	
**条件判断函数：CASE**  

语法: CASE a WHEN b THEN c [WHEN d THEN e]* [ELSE f] END  
返回值: T  
说明：如果a等于b，那么返回c；如果a等于d，那么返回e；否则返回f  

	hive> Select case 100 when 50 then 'tom' when 100 then 'mary' else 'tim' end from iteblog;
	mary
	hive> Select case 200 when 50 then 'tom' when 100 then 'mary' else 'tim' end from iteblog;
	tim  
	
**条件判断函数：CASE**  

语法: CASE WHEN a THEN b [WHEN c THEN d]* [ELSE e] END  
返回值: T  
说明：如果a为TRUE,则返回b；如果c为TRUE，则返回d；否则返回e  

	hive> select case when 1=2 then 'tom' when 2=2 then 'mary' else 'tim' end from iteblog;
	mary
	hive> select case when 1=1 then 'tom' when 2=2 then 'mary' else 'tim' end from iteblog;
	tom  
	

## 集合统计函数  

**个数统计函数: count**  

语法: count(\*), count(expr), count(DISTINCT expr[, expr\_.])  
返回值: int  
说明: count(\*)统计检索出的行的个数，包括NULL值的行；count(expr)返回指定字段的非空值的个数；count(DISTINCT expr[, expr\_.])返回指定字段的不同的非空值的个数  

	hive> select count(*) from iteblog;
	20
	hive> select count(distinct t) from iteblog;
	10  

**总和统计函数: sum**  

语法: sum(col), sum(DISTINCT col)  
返回值: double  
说明: sum(col)统计结果集中col的相加的结果；sum(DISTINCT col)统计结果中col不同值相加的结果  

	hive> select sum(t) from iteblog;
	100
	hive> select sum(distinct t) from iteblog;
	70  

**平均值统计函数: avg**  

语法: avg(col), avg(DISTINCT col)  
返回值: double  
说明: avg(col)统计结果集中col的平均值；avg(DISTINCT col)统计结果中col不同值相加的平均值  

	hive> select avg(t) from iteblog;
	50
	hive> select avg (distinct t) from iteblog;
	30  

**最小值统计函数: min**  

语法: min(col)  
返回值: double  
说明: 统计结果集中col字段的最小值  

	hive> select min(t) from iteblog;
	20  

**最大值统计函数: max**  

语法: maxcol)  
返回值: double  
说明: 统计结果集中col字段的最大值  

	hive> select max(t) from iteblog;
	120  

**非空集合总体变量函数: var_pop**  

语法: var_pop(col)  
返回值: double  
说明: 统计结果集中col非空集合的总体变量（忽略null）    

**非空集合样本变量函数: var_samp**  

语法: var_samp (col)  
返回值: double  
说明: 统计结果集中col非空集合的样本变量（忽略null）  

**总体标准偏离函数: stddev_pop**  

语法: stddev_pop(col)  
返回值: double  
说明: 该函数计算总体标准偏离，并返回总体变量的平方根，其返回值与VAR_POP函数的平方根相同  

**样本标准偏离函数: stddev_samp**  

语法: stddev_samp (col)  
返回值: double  
说明: 该函数计算样本标准偏离  

**中位数函数: percentile**  

语法: percentile(BIGINT col, p)  
返回值: double  
说明: 求准确的第pth个百分位数，p必须介于0和1之间，但是col字段目前只支持整数，不支持浮点数类型     

**中位数函数: percentile**  

语法: percentile(BIGINT col, array(p1 [, p2]…))  
返回值: array<double>  
说明: 功能和上述类似，之后后面可以输入多个百分位数，返回类型也为array<double>，其中为对应的百分位数  

	select percentile(score,&lt;0.2,0.4>) from iteblog； 取0.2，0.4位置的数据  
	
**近似中位数函数: percentile_approx**  

语法: percentile_approx(DOUBLE col, p [, B])   
返回值: double  
说明: 求近似的第pth个百分位数，p必须介于0和1之间，返回类型为double，但是col字段支持浮点类型。参数B控制内存消耗的近似精度，B越大，结果的准确度越高。默认为10,000。当col字段中的distinct值的个数小于B时，结果为准确的百分位数  

**近似中位数函数: percentile_approx**  

语法: percentile_approx(DOUBLE col, array(p1 [, p2]…) [, B])  
返回值: array<double>  
说明: 功能和上述类似，之后后面可以输入多个百分位数，返回类型也为array<double>，其中为对应的百分位数  

**直方图: histogram_numeric**  

语法: histogram_numeric(col, b)  
返回值: array<struct {‘x’,‘y’}>  
说明: 以b为基准计算col的直方图信息  

	hive> select histogram_numeric(100,5) from iteblog;
	[{"x":100.0,"y":1.0}]  
	

## 复合类型构建操作  

**Map类型构建: map**  

语法: map (key1, value1, key2, value2, …)  
说明：根据输入的key和value对构建map类型  

	hive> Create table iteblog as select map('100','tom','200','mary') as t from iteblog;
	hive> describe iteblog;
	t       map<string ,string>
	hive> select t from iteblog;
	{"100":"tom","200":"mary"}  

**Struct类型构建: struct**  

语法: struct(val1, val2, val3, …)  
说明：根据输入的参数构建结构体struct类型  

	hive> create table iteblog as select struct('tom','mary','tim') as t from iteblog;
	hive> describe iteblog;
	t       struct<col1:string ,col2:string,col3:string>
	hive> select t from iteblog;
	{"col1":"tom","col2":"mary","col3":"tim"}  

**array类型构建: array**  

语法: array(val1, val2, …)  
说明：根据输入的参数构建数组array类型  

	hive> create table iteblog as select array("tom","mary","tim") as t from iteblog;
	hive> describe iteblog;
	t       array<string>
	hive> select t from iteblog;
	["tom","mary","tim"]  
	

## 复杂类型访问操作  

**array类型访问: A[n]**  

语法: A[n]  
操作类型: A为array类型，n为int类型  
说明：返回数组A中的第n个变量值。数组的起始下标为0。比如，A是个值为['foo', 'bar']的数组类型，那么A[0]将返回'foo',而A[1]将返回'bar'  

	hive> create table iteblog as select array("tom","mary","tim") as t from iteblog;
	hive> select t[0],t[1],t[2] from iteblog;
	tom     mary    tim  
	
**map类型访问: M[key]**  

语法: M[key]  
操作类型: M为map类型，key为map中的key值  
说明：返回map类型M中，key值为指定值的value值。比如，M是值为{'f' -> 'foo', 'b' -> 'bar', 'all' -> 'foobar'}的map类型，那么M['all']将会返回'foobar'  

	hive> Create table iteblog as select map('100','tom','200','mary') as t from iteblog;
	hive> select t['200'],t['100'] from iteblog;
	mary    tom  

**struct类型访问: S.x**  

语法: S.x  
操作类型: S为struct类型  
说明：返回结构体S中的x字段。比如，对于结构体struct foobar {int foo, int bar}，foobar.foo返回结构体中的foo字段  

	hive> create table iteblog as select struct('tom','mary','tim') as t from iteblog;
	hive> describe iteblog;
	t       struct<col1:string ,col2:string,col3:string>
	hive> select t.col1,t.col3 from iteblog;
	tom     tim  


## 复杂类型长度统计函数  

**Map类型长度函数: size(Map<k .V>)**  

语法: size(Map<k .V>)  
返回值: int  
说明: 返回map类型的长度  

	hive> select size(map('100','tom','101','mary')) from iteblog;
	2  

**array类型长度函数: size(Array\<T\>)**  

语法: size(Array<T>)    
返回值: int    
说明: 返回array类型的长度    

	hive> select size(array('100','101','102','103')) from iteblog;
	4  

**类型转换函数**  

类型转换函数: cast  
语法: cast(expr as <type>)  
返回值: Expected "=" to follow "type"  
说明: 返回转换后的数据类型  

	hive> select cast(1 as bigint) from iteblog;
	1



	
	
[转载](https://www.iteblog.com/)
