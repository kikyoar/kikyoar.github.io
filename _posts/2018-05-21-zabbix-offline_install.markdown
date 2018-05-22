---
layout:     post
title:      "Centos7.4中zabbix3.4.9平台离线搭建"
subtitle:   "zabbix3.4离线安装部署"
date:       2018-05-21
author:     "kikyoar"
header-img: "img/post-bg-linux-version.jpg"
tags:
    - Zabbix
---

## 提前准备

**设置本地yum源**  

	#mount -o loop /usr/local/Centos-7.4-x86_64-dvd.iso /mnt/cdrom
	cd /etc/yum.repos.d/
	mkdir -p bak
	mv *.repo bak/
	vi /etc/yum.repos.d/redhat.repo
	
	[RHEL]
	
	name=Centos7.4        
	baseurl=file:///mnt/cdrom   
	        
	
	gpgcheck=0
	gpgkey=file:///mnt/cdrom/RPM-GPG-KEY-redhat-release
	
	enabled=1  
	
**关闭删除防火墙**  

	systemctl stop firewalld.service  
	systemctl disable firewalld.service   
 
**关闭selinux**

	sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
	grep SELINUX /etc/selinux/config    #确认是否修改成功  
	setenforce 0    

## 安装Nginx  

下载nginx   

	wget http://nginx.org/packages/centos/7/x86_64/RPMS/nginx-1.12.2-1.el7_4.ngx.x86_64.rpm
	rpm -ivh  nginx-1.12.2-1.el7_4.ngx.x86_64.rpm
	systemctl enable nginx
	systemctl start nginx
	systemctl status nginx
	
不关闭nginx，修改配置文件生效方法 nginx -s reload     

## 安装Mysql  

Centos7自带的mysql是mariadb ,我们可以通过如下命令查看 

	yum search mysql|tac
	yum -y install mariadb mariadb-server
	systemctl enable mariadb.service
	systemctl start mariadb.service  
	
**初始化mysql数据库，并配置root用户密码 mysql_secure_installation**  

## 安装PHP  
	
	wget http://cn2.php.net/distributions/php-7.1.11.tar.gz
	tar zxvf php-7.1.11.tar.gz && cd ./php-7.1.11  

安装php依赖包  

	yum install -y libxml2 libxml2-devel openssl openssl-devel bzip2 bzip2-devel libcurl libcurl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel gmp gmp-devel libmcrypt libmcrypt-devel readline readline-devel libxslt libxslt-devel gcc 
  

如果报libmcrypt找不到错误  

	tar zxvf libmcrypt-2.5.8.tar.gz 
	cd libmcrypt-2.5.8 
	./configure
	make
	make install

编译安装php  

	./configure --prefix=/usr/local/php --with-config-file-path=/etc --enable-fpm --with-fpm-user=nginx  --with-fpm-group=nginx --enable-inline-optimization --disable-debug --disable-rpath --enable-shared  --enable-soap --with-libxml-dir --with-xmlrpc --with-openssl --with-mcrypt --with-mhash --with-pcre-regex --with-sqlite3 --with-zlib --enable-bcmath  --with-iconv --with-bz2 --enable-calendar --with-curl --with-cdb --enable-dom --enable-exif --enable-fileinfo --enable-filter --with-pcre-dir --enable-ftp --with-gd --with-openssl-dir  --with-jpeg-dir --with-png-dir --with-zlib-dir --with-freetype-dir --enable-gd-native-ttf --enable-gd-jis-conv --with-gettext --with-gmp --with-mhash --enable-json --enable-mbstring --enable-mbregex --enable-mbregex-backtrack --with-libmbfl --with-onig --enable-pdo --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --with-zlib-dir --with-pdo-sqlite --with-readline --enable-session --enable-shmop --enable-simplexml --enable-sockets  --enable-sysvmsg --enable-sysvsem --enable-sysvshm --enable-wddx --with-libxml-dir --with-xsl --enable-zip --enable-mysqlnd-compression-support --with-pear --enable-opcache  
  

PHP编译时错误：Don’t know how to define struct flock on this system, set –enable-opcache=no.可能是/usr/local/lib下的库文件没有加载，可如下操作：  
	
	vim /etc/ld.so.conf.d/local.conf     # 编辑库文件（该文件可能不存在，不存在则创建一个新的）
	/usr/local/lib                       # 添加该行
	/usr/local/lib64                     # 64位系统的除了添加上一行，还需要添加此行
	:wq                                  # 保存退出
	ldconfig -v                          # 使之生效  

编译与安装,需要一段时间慢慢等候   

	make && make install  
	
添加 PHP 命令到环境变量 vi /etc/profile,在末尾加入   
	
	PATH=$PATH:/usr/local/php/bin
	
	export PATH
	source /etc/profile
	echo $PATH
	php -v
	
配置php-fpm 
	
	cp ./php.ini-production /etc/php.ini  #从php源代码中拷贝php.ini-production 到并命名为 /etc/php.ini
	cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf
	cp /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/php-fpm.d/www.conf
	cp sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
	chmod +x /etc/init.d/php-fpm
	
	sed -i "s@date.timezone =@date.timezone = Asia/Shanghai@g" /etc/php.ini
	sed -i "s@max_execution_time = 30@max_execution_time = 300@g" /etc/php.ini
	sed -i "s@post_max_size = 8M@post_max_size = 32M@g" /etc/php.ini
	sed -i "s@max_input_time = 60@max_input_time = 300@g" /etc/php.ini
	sed -i "s@;memory_limit = 128M@memory_limit = 128M@g" /etc/php.ini
	sed -i "s@;mbstring.func_overload = 0@mbstring.func_overload = 2@g" /etc/php.ini
	
	/etc/init.d/php-fpm start    
	
配置nginx,绑定服务器 

	vi /etc/nginx/conf.d/default.conf  

	server{
	    listen 8888;
	    server_name  127.0.0.1;
	    root /var/www/html/;  # 该项要修改为你准备存放相关网页的路径
	    location / {
	        index  index.php index.html index.htm;
	        #如果请求既不是一个文件，也不是一个目录，则执行一下重写规则
	        if (!-e $request_filename)
	        {
	            #地址作为将参数rewrite到index.php上。
	            rewrite ^/(.*)$ /index.php/$1;
	            #若是子目录则使用下面这句，将subdir改成目录名称即可。
	            #rewrite ^/subdir/(.*)$ /subdir/index.php/$1;
	        }
	    }
	    #proxy the php scripts to php-fpm
	    location ~ \.php {
	        include fastcgi_params;
	        ##pathinfo支持start
	        #定义变量 $path_info ，用于存放pathinfo信息
	        set $path_info "";
	        #定义变量 $real_script_name，用于存放真实地址
	        set $real_script_name $fastcgi_script_name;
	        #如果地址与引号内的正则表达式匹配
	        if ($fastcgi_script_name ~ "^(.+?\.php)(/.+)$") {
	            #将文件地址赋值给变量 $real_script_name
	            set $real_script_name $1;
	            #将文件地址后的参数赋值给变量 $path_info
	            set $path_info $2;
	        }
	        #配置fastcgi的一些参数
	        fastcgi_param SCRIPT_FILENAME $document_root$real_script_name;
	        fastcgi_param SCRIPT_NAME $real_script_name;
	        fastcgi_param PATH_INFO $path_info;
	        ###pathinfo支持end
	        fastcgi_intercept_errors on;
	        fastcgi_pass   127.0.0.1:9000;
	    }

	    location ^~ /data/runtime {
	        return 404;
	    }
	
	    location ^~ /application {
	        return 404;
	    }
	
	    location ^~ /simplewind {
	        return 404;
	    }
	}  

 
重启nginx  

	nginx -s reload  

创建测试php文件   
  vi /var/www/html/info.php

    <?php
        phpinfo();
    ?>

访问 http://127.0.0.1/info.php    

## 安装zabbix-server

	yum install mysql-devel
	yum -y install net-snmp-devel libxml2-devel libcurl-deve libevent libevent-devel  

Yum安装OpenIPMI并下载OpenIPMI-devel-2.0.19-15.el7.x86_64.rpm , libevent-devel-2.0.21-4.el7.x86_64.rpm     

	# yum install OpenIPMI
	# rpm -ivh OpenIPMI-devel-2.0.19-15.el7.x86_64.rpm  
	systemctl start ipmi
	systemctl enable ipmi
	rpm -ivh /opt/libevent-devel-2.0.21-4.el7.x86_64.rpm  

编译安装zabbix  

	./configure --prefix=/usr/local/zabbix-3.4.9 --enable-server --enable-agent --enable-java --with-mysql --enable-ipv6 --with-net-snmp --with-libcurl --with-libxml2 --with-openipmi --with-unixodbc --with-openssl
 
	make && make install    

登录数据库  

	create database zabbix character set utf8 collate utf8_bin;
	
	grant all privileges on zabbix.* to 'zabbix'@'%' identified by 'zabbixpass';
	
	flush privileges;
	
	
	cd /usr/local/zabbix/database/mysql
	
	mysql -uzabbix -p
	
	use zabbix;
	source /usr/local/zabbix/database/mysql/schema.sql;
	source /usr/local/zabbix/database/mysql/images.sql;
	source /usr/local/zabbix/database/mysql/data.sql;  

配置zabbix  

	ln -s /usr/local/zabbix/etc/ /etc/zabbix
	ln -s /usr/local/zabbix/bin/* /usr/bin/
	ln -s /usr/local/zabbix/sbin/* /usr/sbin/
	
	
	cd /usr/local/zabbix/misc/init.d/fedora/core
	cp zabbix_* /etc/init.d/
	sed -i "s@BASEDIR=/usr/local@BASEDIR=/usr/local/zabbix@g" /etc/init.d/zabbix_server
	sed -i "s@BASEDIR=/usr/local@BASEDIR=/usr/local/zabbix@g" /etc/init.d/zabbix_agentd
	chmod 755 /etc/init.d/zabbix_*

	cd /usr/local/zabbix
	mkdir logs
	chown zabbix:zabbix logs
	
	sed -i "s@Server=127.0.0.1@Server=127.0.0.1,192.168.100.135@g" /etc/zabbix/zabbix_agentd.conf
	sed -i "s@ServerActive=127.0.0.1@ServerActive=192.168.100.135:10051@g" /etc/zabbix/zabbix_agentd.conf
	sed -i "s@^# UnsafeUserParameters=0@UnsafeUserParameters=1@g" /etc/zabbix/zabbix_agentd.conf
	sed -i "s@tmp/zabbix_agentd.log@usr/local/zabbix/logs/zabbix_agentd.log@g" /etc/zabbix/zabbix_agentd.conf
	
	cp -r /usr/local/zabbix/frontends/php/ /var/www/html/zabbix/
	chown -R apache.apache /var/www/html/zabbix/
	
	systemctl start zabbix_server
	systemctl start zabbix_agend  

## zabbix转中文

	下载标准中文字体simkai.tff，并上传至/var/www/html/zabbix/fonts/
	
	修改php页面指定的字体文件：
	
	vim /var/www/html/zabbix/include/defines.inc.php   
	
	define('ZBX_GRAPH_FONT_NAME',   ' DejaVuSans')改为define('ZBX_GRAPH_FONT_NAME',  'simkai');
	
	http: service nginx restart  


## 配置snmp

	vi /etc/snmp/snmpd.conf
	com2sec notConfigUser  default   hadoop           //将public改为hadoop
	view    systemview    included   .1                  //手动新增加这行
	proc mountd                    //找到配置，把注释去掉
	proc ntalkd 4                    //找到配置，把注释去掉
	proc sendmail 10 1               //找到配置，把注释去掉
	disk / 10000                    //找到配置，把注释去掉
	load 12 14 14                   //找到配置，把注释去掉
	systemctl start snmpd.service 
	systemctl enable snmpd.service     

## 安装zabbix_agent 

	yum -y install net-snmp-devel libxml2-devel libcurl-deve libevent libevent-devel
	scp  /usr/local/zabbix-3.4.9.tar.gz   root@192.168.100.11:/usr/local
	mv zabbix-3.4.9 zabbix
	cd zabbix
	./configure --prefix=/usr/local/zabbix --enable-agent   
	make && make install
	
	groupadd zabbix
	useradd -g zabbix -m zabbix
	mkdir logs
	chown zabbix:zabbix logs
	cp misc/init.d/fedora/core/zabbix_agentd  /etc/init.d/
	chmod 755 /etc/init.d/zabbix_agentd
	
	sed -i "s@BASEDIR=/usr/local@BASEDIR=/usr/local/zabbix@g" /etc/init.d/zabbix_agentd
	ln -s /usr/local/zabbix/etc/ /etc/zabbix
	ln -s /usr/local/zabbix/bin/*  /usr/bin/
	ln -s /usr/local/zabbix/sbin/*  /usr/sbin/
	
	vi /etc/zabbix/zabbix_agentd.conf
	Server=192.168.100.135
	ServerActive=192.168.100.135
	LogFile=/usr/local/zabbix/logs/zabbix_agentd.log
	UnsafeUserParameters=1
	Hostname=gsunicom-nifi-01
	/etc/init.d/zabbix_agentd start  
	 

## 客户端SNMP配置

	vi /etc/snmp/snmpd.conf
	com2sec notConfigUser  default   hadoop           //将public改为hadoop
	view    systemview    included   .1                  //手动新增加这行
	systemctl start snmpd.service 
	systemctl enable snmpd.service   
	
以下为配置图，**切记一定要改宏**
![zabbix snmp配置](http://kikyoar.com/img/zabbix_offline_install_snmp01.png)
![zabbix snmp配置](http://kikyoar.com/img/zabbix_offline_install_snmp02.png)



	
		

	
