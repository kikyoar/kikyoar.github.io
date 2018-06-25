---
layout:     post
title:      "Shell脚本实现FTP上传下载文件"
subtitle:   "Linux下使用Shell脚本实现FTP自动上传和下载文件"
date:       2018-06-25
author:     "kikyoar"
header-img: "img/post-bg-linux-version.jpg"
tags:
    - Linux
---   

**登录FTP实现下载文件功能**  

FTP服务器：10.11.10.11   
FTP路径：Down   
本地路径： ./   
将文件从FTP下载到本地的脚本如下：  

## 命令解释  

**登录FTP**

	ftp -i -n 10.11.10.11 << EOF  
	
<< 是使用即时文件重定向输入   
EOF是即时文件的标志它必须成对出现，以标识即时文件的开始和结尾   
ftp常见的几个标志有：  

	-d：使用调试模式，但必须编辑 /etc/syslog.conf 文件并添加以下中的一项：user.info FileName 或 user.debug FileName 
	-g：禁用文件名中的元字符拓展，即取消全局文件名
	-i ：关闭多文件传输中的交互式提示 
	-n：防止在起始连接中的自动登录。否则， ftp 命令会搜索 $HOME/.netrc 登录项，该登录项描述了远程主机的登录和初始化过程 
	-v：显示远程服务器的全部响应，并提供数据传输的统计信息，即在程序运行时显示详细的处理信息  
	
**输入FTP用户名和密码**  

	user ftpuser ftppwd  

ftpuser：登录FTP时的用户名   
ftppwd：登录FTP时的密码  

**通过binary命令传输文件**  

	binary  
	
FTP文件传输类型有： ascii、binary、ebcdic、image、local M 和 tenex  

	– ascii：将文件传输类型设置为网络 ASCII。此类型为缺省值，即默认使用ascii方式进行传输 
	– binary：将文件传输类型设置为二进制映像。需要使用binary方式传输的文件类型有ISO文件、可执行文件、压缩文件、图片等。此类型可能比 ASCII 传送更有效
	– ebcdic：将文件传输类型设为 EBCDIC 
	– image：将文件传输类型设置为二进制映像。此类型可能比 ASCII 传送更有效 
	– local M：将文件传输类型设置为本地。M 参数定义每计算机字位的十进制数。此参数没有缺省值 
	– tenex：将文件传输类型设为 TENEX 机器需要的类型  
	
**切换散列符号 （#） 打印**

	hash

当用get或put命令传送一个数据块时，让FTP显示一个#，这是看得见的确定数据在传输的信号，在用户不确信网络是否工作时有用。当传输很大的文件时，如果FTP已显示这种信息，表示传输正在进行。hash命令是一个布尔变量式的命令，用hash命令打开显示#开关，再用hash命令关闭显示  

**切换目录**   

到FTP上对应路径（这里表示进入Dwon文件夹下）：  

	cd ./Down  
	
到本地的对应路径（这里表示在当前文件夹下）：  

	lcd ./  
	
**切换交互式提示**  

	prompt  
	
使用mget或mput时，prompt命令让FTP在传输每个文件前进行提示，这样防止覆盖已有的文件。若发出prompt命令时已经启动了提示，FTP将把提示关掉，此时再传输所有的文件则不会有任何提问  

**下载文件**  

下载多个文件：   
格式：mget [remote-files]   
例如：获取远端所在文件夹下所有文件  

	mget *
	#或
	mget *.*  
	
**注：mget . 每下载一个文件，都会有提示。如果要除掉提示，则在mget . 命令前先执行:prompt off**  

下载单个文件：   
格式：get [remote-file] [local-file]   
例如：获取远端FTP上的a.txt文件  

	get a.txt  
	
**上传文件**  

上传多个文件：   
格式：mput local-files   
例如：将所在文件夹下所有文件上传到FTP上  

	mput *  
	
上传单个文件：   
格式：put local-file [remote-file]   
例如：将本地a.txt文件上传到远端FTP上  

	put a.txt  
	
**断开连接**  

	bye  
	
结束文件传输会话并退出 ftp 命令，与quit命令相同  

**所有命令详解：**

	FTP的命令行格式为：ftp -v -d -i -n -g [主机名]，其中
	-v 显示远程服务器的所有响应信息；
	-i 传送多个文件时禁用交互提示
	-n 在建立初始连接后禁止自动登录功能 
	.n etrc文件；
	-d 使用调试方式；
	-g 取消全局文件名
	
	ftp使用的内部命令如下(中括号表示可选项):
	1.![cmd[args]]：在本地机中执行交互shell，exit回到ftp环境，如：!ls*.zip.
	2.$ macro-ame[args]：执行宏定义macro-name.
	3.account[password]：提供登录远程系统成功后访问系统资源所需的补充口令。
	4.append local-file[remote-file]：将本地文件追加到远程系统主机，若未指定远程系统文件名，则使用本地文件名。
	5.ascii：使用ascii类型传输方式。
	6.bell：每个命令执行完毕后计算机响铃一次。
	7.bin：使用二进制文件传输方式。
	8.bye：退出ftp会话过程。
	9.case：在使用mget时，将远程主机文件名中的大写转为小写字母。
	10.cd remote-dir：进入远程主机目录。
	11.cdup：进入远程主机目录的父目录。
	12.chmod mode file-name：将远程主机文件file-name的存取方式设置为mode，如：chmod 777 a.out。
	13.close：中断与远程服务器的ftp会话(与open对应)。
	14.cr：使用asscii方式传输文件时，将回车换行转换为回行。
	15.delete remote-file：删除远程主机文件。
	16.debug[debug-value]：设置调试方式，显示发送至远程主机的每条命令，如：deb up 3，若设为0，表示取消debug。
	17.dir[remote-dir][local-file]：显示远程主机目录，并将结果存入本地文件local-file。
	18.disconnection：同close。
	19.form format：将文件传输方式设置为format，缺省为file方式。
	20.get remote-file[local-file]：将远程主机的文件remote-file传至本地硬盘的local-file。
	21.glob：设置mdelete，mget，mput的文件名扩展，缺省时不扩展文件名，同命令行的-g参数。
	22.hash：每传输1024字节，显示一个hash符号(#)。
	23.help[cmd]：显示ftp内部命令cmd的帮助信息，如：help get。
	24.idle[seconds]：将远程服务器的休眠计时器设为[seconds]秒。
	25.image：设置二进制传输方式(同binary)。
	26.lcd[dir]：将本地工作目录切换至dir。
	27.ls[remote-dir][local-file]：显示远程目录remote-dir，并存入本地文件local-file。
	28.macdef macro-name：定义一个宏，遇到macdef下的空行时，宏定义结束。
	29.mdelete[remote-file]：删除远程主机文件。
	30.mdir remote-files local-file：与dir类似，但可指定多个远程文件，如：mdir *.o.*.zipoutfile
	31.mget remote-files：传输多个远程文件。
	32.mkdir dir-name：在远程主机中建一目录。
	33.mls remote-file local-file：同nlist，但可指定多个文件名。
	34.mode[modename]：将文件传输方式设置为modename，缺省为stream方式。
	35.modtime file-name：显示远程主机文件的最后修改时间。
	36.mput local-file：将多个文件传输至远程主机。
	37.newer file-name：如果远程机中file-name的修改时间比本地硬盘同名文件的时间更近，则重传该文件。
	38.nlist[remote-dir][local-file]：显示远程主机目录的文件清单，并存入本地硬盘的local-file。
	39.nmap[inpattern outpattern]：设置文件名映射机制，使得文件传输时，文件中的某些字符相互转换，如：nmap $1.$2.$3[$1，$2].[$2，$3]，则传输文件a1.a2.a3时，文件名变为a1，a2。该命令特别适用于远程主机为非UNIX机的情况。
	40.ntrans[inchars[outchars]]：设置文件名字符的翻译机制，如ntrans 1R，则文件名LLL将变为RRR。
	41.open host[port]：建立指定ftp服务器连接，可指定连接端口。
	42.passive：进入被动传输方式。
	43.prompt：设置多个文件传输时的交互提示。
	44.proxy ftp-cmd：在次要控制连接中，执行一条ftp命令，该命令允许连接两个ftp服务器，以在两个服务器间传输文件。第一条ftp命令必须为open，以首先建立两个服务器间的连接。
	45.put local-file[remote-file]：将本地文件local-file传送至远程主机。
	46.pwd：显示远程主机的当前工作目录。
	47.quit：同bye，退出ftp会话。
	48.quote arg1，arg2...：将参数逐字发至远程ftp服务器，如：quote syst.
	49.recv remote-file[local-file]：同get。
	50.reget remote-file[local-file]：类似于get，但若local-file存在，则从上次传输中断处续传。
	51.rhelp[cmd-name]：请求获得远程主机的帮助。
	52.rstatus[file-name]：若未指定文件名，则显示远程主机的状态，否则显示文件状态。
	53.rename[from][to]：更改远程主机文件名。
	54.reset：清除回答队列。
	55.restart marker：从指定的标志marker处，重新开始get或put，如：restart 130。
	56.rmdir dir-name：删除远程主机目录。
	57.runique：设置文件名唯一性存储，若文件存在，则在原文件后加后缀..1，.2等。
	58.send local-file[remote-file]：同put。
	59.sendport：设置PORT命令的使用。
	60.site arg1，arg2...：将参数作为SITE命令逐字发送至远程ftp主机。
	61.size file-name：显示远程主机文件大小，如：site idle 7200。
	62.status：显示当前ftp状态。
	63.struct[struct-name]：将文件传输结构设置为struct-name，缺省时使用stream结构。
	64.sunique：将远程主机文件名存储设置为唯一(与runique对应)。
	65.system：显示远程主机的操作系统类型。
	66.tenex：将文件传输类型设置为TENEX机的所需的类型。
	67.tick：设置传输时的字节计数器。
	68.trace：设置包跟踪。
	69.type[type-name]：设置文件传输类型为type-name，缺省为ascii，如：type binary，设置二进制传输方式。
	70.umask[newmask]：将远程服务器的缺省umask设置为newmask，如：umask 3。
	71.user user-name[password][account]：向远程主机表明自己的身份，需要口令时，必须输入口令，如：user anonymous my@email。
	72.verbose：同命令行的-v参数，即设置详尽报告方式，ftp服务器的所有响应都将显示给用户，缺省为on.
	73.?[cmd]：同help  
	

## 实例 
实现功能：根据已有目录名抓取远程目录文件，并按照FTP目录新建文件夹，下载完成后，压缩传送到另一台服务器

	#! /bin/bash
	# FTP name match && receive directory && deliver other servers
	# Please run the script in the root directory
	# yuhao.zhang
	# version 0.1
	# date 2018.06.21
	
	# 脚本说明：
	
	# 1、ENID提前准备好，ENID必须先用cat -A查看，去除<tab> 与断行字符后放到root目录下，不然新建的文件夹会有乱码
	# 2、该脚本和ENID置于/root/下
	# 3、传送的服务器之间必须是免密
	# 4、以上务必完成后再执行脚本
	
	
	#定义变量
	remoteIp1="192.168.1.2"
	ftpuser="用户名"   #自行修改
	passwd="密码"   #自行修改
	remoteDir="l3fw_mr/kpi_import"
	one_day_before=`date -d '1 days ago' +%Y%m%d`
	mkdir /root/test_beijing
	mkdir /root/test_beijing/$one_day_before
	
	#ENID是需要下载的目录名，提前准备好
	matchDir="/root/ENID"     #ENID必须先用cat -A查看，去除<tab> 与断行字符后放到这
	
	echo "**********************************************"
	echo "开始下载第一台服务器192.168.1.2"
	#前一天
	for rname in $remoteIp1
	do
	    ftp -i -n $rname <<EOF | awk '/^d/{print one_day_before $NF}' >> $remoteIp1_$one_day_before.log
	    user ${ftpuser} ${passwd}
	    cd ${remoteDir}
	    cd $one_day_before
	    ls -l
	    bye
	EOF
	done
	
	
	grep -wf $remoteIp1_$one_day_before.log $matchDir > match_$one_day_before.log
	
	#for i in $(cat 192.168.1.2_$one_day_before.log); do
	#  grep "\<$i\>" $matchDir > match.log
	#done
	
	
	for line in $(cat match_$one_day_before.log)
	do
	    cd test_beijing
	    cd $one_day_before
	    mkdir $line
	    mm=$line
	    ftp -v -n $remoteIp1 <<EOF
	    user ${ftpuser} ${passwd}
	    cd ${remoteDir}
	    cd $one_day_before
	    cd $line
	    lcd /root/test_beijing/$one_day_before/$mm
	    prompt off
	    mget *
	    bye
	EOF
	echo "one_day_before download from ftp successfully"
	    # DIR="//192.168.1.2:21/l3fw_mr/kpi_import/20180621/$rname/*"
	    # wget --ftp-user=richuser --ftp-password=richr00t ftp:$DIR -o wget.log -O /root/test_beijing
	done
	
	sleep 30s
	echo "开始传送"
	tar -cvf test_beijing.tar test_beijing/
	scp test_beijing.tar root@192.168.11.2:/fastdata/FINISHED_LOCATION/beijing/LZ/


 

