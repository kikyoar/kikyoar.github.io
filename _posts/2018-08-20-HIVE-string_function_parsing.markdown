---
layout:     post
title:      "Hive常用字符串函数"
subtitle:   "数据库开发必备"
date:       2018-08-20
author:     "kikyoar"
header-img: "img/post-bg-hadoop-version.jpg"
tags:
    - Hadoop
---  


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
	
	

	
	
[转载](https://www.iteblog.com/)
