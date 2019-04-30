---
layout:     post
title:      "Linux下部署FTP服务器详解"
subtitle:   "vsftp服务的多种配置方法"
date:       2019-04-30
author:     "kikyoar"
header-img: "img/post-bg-linux-version.jpg"
tags:
    - Linux
---  

FTP 服务是 client/server （简称C/S ）模式，基于 FTP 协议实现 FTP 文件对外共享及传输的软件称之为 FTP 服务器源端，客户端程序基于 FTP 协议，则称之为 FTP 客户端，FTP 客户端可以向 FTP 服务器上传、下载文件


### FTP传输模式  

FTP 客户端与服务器端有两种传输模式，分别是 FTP 主动模式、FTP 被动模式  

- 主动FTP：  
   　　命令连接：客户端 >1023端口 -> 服务器 21端口  
   　　数据连接：客户端 >1023端口 <- 服务器 20端口  

- 被动FTP：  
   　　命令连接：客户端 >1023端口 -> 服务器 21端口  
   　　数据连接：客户端 >1023端口 -> 服务器 >1023端口  
   　　

###  Vsftpd服务器安装配置  

- 在 shell 命令行执行如下命令:  

		yum install vsftpd* -y

- vsftpd 安装后的配置文件路径、启动 Vsftpd 服务及查看进程是否启动
		
		rpm -ql vsftpd | more
		systemctl restart vsftpd.service
		ps -ef | grep vsftpd 
		
- vsftpd.conf 默认配置文件详解如下：  

	    anonymous_enable= YES	  #开启匿名用户访问
	    local_enable= YES	  #启用本地系统用户访问
	    write_enable= YES	  #本地系统用户写人权限
	    local_umask=022	  #本地用户创建文件及目录默认权限掩码
	    dirmessage_enable= YES	 #打印目录显示信息，通常用于用户第一次访问目录时，信息提示
	    xferlog_enable= YES 	#启用上传，下载日志记录
	    connect_from_port_20 =YES 	#FTP使用20端口进行数据传输
	    xferlog_std_format= YES    #日志文件将根据 xferlog 的标准格式写人
	    listen=NO	 	#Vsftpd 不以独立的服务启动，通过 Xinetd 服务管理，建议改成 YES
	    listen_ipv6 =YES  #启用 IPv6 监昕
	    pam_service_name= vsftpd	#登录 FTP 服务器，依据 /etc/pam.d/vsftpd 中内容进行认证
	    userlist_ enable= YES  #vsftpd.user_list和ftpusers 配置文件里用户禁止访问 FTP
	    tcp_wrappers= YES  #设置 Vsftpd与tcp wrapper 结合进行主机的访问控制,Vsftpd服务器检查 /etc/hosts.allow和/etc/hosts.deny 中的设置来决定请求连接的主机，是否允许访问该FTP 服务器


	- FTP 主被动模式的选择，默认为主动模式，设置为被动模式的方法如下：  

			pasv_enable= YES 
			pasv_min_port=6000
			pasv_max_port=60100



	- 配置vsftpd服务为主动模式:

			connect_from_port_20=YES
			pasv_enable=NO
			
### Vsftpd服务器匿名用户配置  

如需关闭 FTP 匿名用户访问，需修改配置文件 `/etc/vsftpd/vsftpd.conf` ，将 anonymous_ enable=YES 修改为 anonymous_ble=NO ，重启 Vsftpd 服务即可  

如果允许匿名用户能够上传、下载、删除文件，需在 /etc/vsftpd/vsftpd.conf 配置文件  

加入以下代码，详解如下：  

	anon_upload_enable= YES ：允许匿名用户上传文件
	anon_mkdir_write_enable= YES ：允许匿名用户创建目录
	anon_other_write_enable= YES ：允许匿名用户其他写入权限

由于默认 Vsftpd 匿名用户有两种： anonymous和ftp ，所以匿名用户如果需要上传文件、删除及修改等权限，需要 Vsftpd 用户对 /var/ftp/pub 目录有写人权限,使用 chown或者chmod 任意一种命令均可设置权限，具体设置命令如下：

	chown -R ftp pub/
	chmod o+w pub/   
	
### Vsftpd服务器系统用户配置  

- Linux 系统中创建系统用户 jfedul，jfedu2 ，分别设置密码为 123456

		useradd jfedul 
		useradd jfedu2 
		echo 123456 | passwd —stdin jfedul 
		echo 123456 | passwd  —stdin jfedu2 

- 修改 vsftpd. conf 配置文件代码如下：  

		anonymous_enable = NO
		local_enable = YES
		write_enable = YES
		local_umask = 022
		dirmessage_enable = YES
		xferlog_enable = YES
		connect_from_port_20 = YES
		xferlog_std_format = YES
		listen= NO
		listen_ipv6 =YES
		pam_service_name= vsftpd
		user_list_enable= YES
		tcp_wrappers =YES   
		
### Vsftpd服务器虚拟用户配置  

Vsftpd 基于系统用户访问 FTP 服务器，系统用户越多越不利于管理，而且不利于系统安全 ，为了能更加安全使用 Vsftpd ，可以使用 Vsftpd 虚拟用户方式  
Vsftpd 虚拟用户原理为虚拟用户没有实际的真实系统用户，而是通过映射到其中一个真实用户以及设置相应的权限来实现访问验证，虚拟用户不能登录 Linux 系统，从而让系统更加的安全可靠  

Vsftpd 虚拟用户企业案例配置步骤如下：  

- 安装 Vsftpd 虚拟用户需要用到的软件及认证模块  

yum install pam* libdb-utils libdb --skip -broken - y  

- 创建虚拟用户临时文件 /etc/vsftpd/ftpusers.txt，新建虚拟用户和密码，其中jfeduOO1， jfedu002 为虚拟用户名， 123456 为密码，如果有多个用户，依此格式填写即可  

		jfedu001
		123456
		jfedu002
		123456
		
- 生成 Vsftpd 虚拟用户数据库认证文件，设置权限为 700  

		db_load -T -t hash -f /etc/vsftpd/ftpusers.txt /etc/vsftpd/vsftpd_login.db  
		chmod 700 /etc/vsftpd/vsftpd_login.db  
		
		
- 配置 PAM 认证文件，/etc/pam.d/vsftpd `行首`加入如下两行代码：  

		auth required pam_userdb.so db= /etc/vsftpd/vsftpd_login
		account required pam_userdb.so db= /etc/vsftpd/vsftpd_login  
		
- Vsftpd 虚拟用户需要映射到一个系统用户，该系统用户不需要密码，也不需要登录，主要用于虚拟用户映射使用，创建用户命令如下：  

		useradd - s /sbin/nologin ftpuser  
		
- 完整的 vsftpd.conf 配置文件代码如下：  

		anonymous_enable = YES
		local_enable = YES
		write_enable = YES
		local_umask = 022
		dirmessage_enable = YES
		xferlog_enable = YES
		connect_from_port_20 = YES
		xferlog_std_format = YES
		listen= NO
		listen_ipv6 =YES
		user_list_enable= YES
		tcp_wrappers =YES	
		# config virtual user FTP 	
		pam_service_name= vsftpd
		guest_enable= YES 
		guest_username = ftpuser
		user_config_dir = /etc/vsftpd/vsftpd_user_conf 
		virtual_use_local_privs =YES  
		
- Vsftpd 虚拟用户配置文件参数详解如下：

		pam_service_name= vsftpd ：虚拟用户启用pam认证
		guest_enable= YES ：启用虚拟用户
		guest_username= ftpuser ：映射虚拟用户至系统用户ftpuser
		user_config_dir = /etc/vsftpd/vsftpd_user_conf ：设置虚拟用户配置文件所在的目录
		virtual_use_local_privs= YES ：虚拟用户使用与本地用户相同的权限
		
- 至此，所有虚拟用户共同使用/home/ftpuser 主目录实现文件上传与下载，可以在/etc/vsftpd/vsftpd\_user\_conf 目录创建虚拟用户各自的配置文件，创建虚拟用户配置文件主目录，代码如下：  

		mkdir -p /etc/vsftpd/vsftpd_user_conf/ 
		
- 以下分别为虚拟用户 fedu001 jfedu002 创建配置文件  

  vim /etc/vsftpd/vsftpd_user_conf/jfedu001,同时创建私有的虚拟目录，代码如下： 
		
		local_root=/home/ftpuser/jfeduOOl
		write_enable= YES
		anon_world_readable_only =YES
		anon_upload_enable= YES
		anon_mkdir_write_enable= YES
		anon_other_write_enable= YES 


  vim /etc/vsftpd/vsftpd_user_conf/jfedu002,同时创建私有的虚拟目录，代码如下： 
		
		local_root=/home/ftpuser/jfeduOO2
		write_enable= YES
		anon_world_readable_only =YES
		anon_upload_enable= YES
		anon_mkdir_write_enable= YES
		anon_other_write_enable= YES 
		
  虚拟用户配置文件内容详解如下：  
  
		local_root= /home/ftpuser/jfeduOO2: jfedu002 虚拟用户配置文件路径
		write_enable= YES ：允许登录用户有写权限
		anon_world_readable_only =YES ：允许匿名用户下载，然后读取文件
		anon_upload_enable= YES ：允许匿名用户上传文件权限，只有在 write_enable=YES 时该参数才生效
		anon_mkdir_write_enable= YES ：允许匿名用户创建目录，只有在 write_enable=YES 时该参数才生效
		anon_other_write_enable= YES ：允许匿名用户其他权限，例如删除、重命名等  
		
- 创建虚拟用户各自虚拟目录，代码如下：

		mkdir -p /home/ftpuser/{jfedu001, jfedu002}
		chown -R ftpuser:ftpuser /home/ftpuser 
