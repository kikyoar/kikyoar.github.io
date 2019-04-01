---
layout:     post
title:      "linux下rm命令改造-把文件删除到回收站"
subtitle:   "防止误删操作，Linux加固"
date:       2019-04-01
author:     "kikyoar"
header-img: "img/post-bg-linux-version.jpg"
tags:
    - Linux
---  

`rm -rf ./*`  
来删除当前文件夹文件。不幸的是你敲成了这样： 
 
`rm -rf /*`  
**那就赶紧跑路吧**  

具体操作如下：
  
- 在/opt新建.trash回收站：  

		mkdir /opt/.trash

- 在.bashrc末尾中添加如下配置：

		# 修改rm
		alias rm=trash  
		alias rl='ls /opt/.trash'
		alias ur=cleartrash
		trash()
		{
		  mv $@ /opt/.trash/
		}
		cleartrash()
		{
		    read -p "clear sure?[n]" confirm
		    [ $confirm == 'y' ] || [ $confirm == 'Y' ]  && /usr/bin/rm -rf /opt/.trash/*
		}

**修改完毕后，使用`source .bashrc`更新下，然后你就可以使用如下命令了：**  

* rm: 删除文件或目录到回收站
* rl: 查看回收站内容
* ur: 清空回收站  


如果手动删除嫌麻烦，可以做个定时任务，规则参考：

- 删除三天前的回收站文件或目录
- 删除大于800M的回收站目录


