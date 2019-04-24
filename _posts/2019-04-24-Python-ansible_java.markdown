layout:     post
title:      "使用Ansible自动配置JDK环境"
subtitle:   "Ansible roles的应用实践"
date:       2019-04-24
author:     "kikyoar"
header-img: "img/post-bg-python-version.jp"
tags:   - Python

[源代码放置于Github](<https://github.com/kikyoar/Hadoop-Automated-scripts>)

> 本项目是自动化部署安装JAVA  
> 当前版本是：v1.0
>
> 源码目录树为：
>
> .
> ├── defaults
> │   └── java_roles.yaml
> ├── files
> ├── handlers
> ├── readme.md
> ├── tasks
> │   ├── check.yaml
> │   ├── ln_jdk.yaml
> │   ├── main.yaml
> │   ├── set_var.yaml
> │   ├── unarchive_jdk.yaml
> │   └── uninstall_openjdk.yaml
> ├── templetes
> └── vars
>     └── main.yaml

## 所需软件

`以下版本均可自行定义`  

| 软件名称 | 软件版本      |
| :------- | :------------ |
| ansible  | ansible 2.7.0 |
| java     | 1.8.0_181     |

## 操作指南

- 可利用yum安装ansible，有外网环境的话安装Python3.6，亦可pip3安装ansible(这样ansible版本较高)

- 配置ansible：ansible目录应该默认在/etc/ansible下，此目录下必须有如下文件

  - ansible.cfg,[github下载](https://raw.githubusercontent.com/ansible/ansible/devel/examples/hosts)
  - hosts,[github下载](https://raw.githubusercontent.com/ansible/ansible/devel/examples/hosts)
  - roles目录

- 设置SSH免密登录安装java的主机

- 将压缩包java_install.zip解压拷贝至roles目录下

- 将需要安装java的主机IP地址填写于hosts文件required_java下

  [required_java]
  	192.168.6.62

- 此压缩包中已自带java(jdk-8u181-linux-x64.tar.gz)--由于文件太大未上传（雾）,如果有新的java版本需求，请上传至files目录下替换该文件，然后修改vars目录下main.yaml中的value值

  java_packages: jdk-8u181-linux-x64.tar.gz
  	java_install_name: jdk1.8.0_181

- 将defaults目录下的java_roles.yaml拷贝至/etc/ansible/目录下

- 在/etc/ansible/执行命令，如若未发生报错，执行下一步

  [root@ceshi-1 ansible]# ansible-playbook -C java_roles.yaml

- 在/etc/ansible/执行命令

  [root@ceshi-1 ansible]# ansible-playbook java_roles.yaml

- 以上安装完成



## 研究点  

##### ansible register

uninstall_openjdk.yaml这个yaml文件起先是这样写的：



```yaml
- name: uninstall jdk
  shell: rpm -qa | grep jdk | xargs rpm -e --nodeps
  when: jdk_result is succeeded

- name: uninstall java
  shell: rpm -qa | grep java | xargs rpm -e --nodeps
```

但是在执行过程中报错为[rpm：未给出要擦除的软件包](),后才采用下面这种格式：

```yaml
- name: visit jdk
  shell: rpm -qa | grep jdk
  register: jdk_result
  ignore_errors: True

- name: visit java
  shell: rpm -qa | grep java
  register: java_result
  ignore_errors: True

- name: uninstall jdk
  shell: rpm -qa | grep jdk | xargs rpm -e --nodeps
  when: jdk_result is succeeded

- name: uninstall java
  shell: rpm -qa | grep java | xargs rpm -e --nodeps
  when: jdk_result is succeeded
```

其中ansible register 这个功能非常有用，当我们需要判断对执行了某个操作或者某个命令后，如何做相应的响应处理（执行其他 ansible 语句），则一般会用到register  

**举个例子**：

我们需要判断sda6是否存在，如果存在了就执行一些相应的脚本，则可以为该判断注册一个register变量，并用它来判断是否存在，存在返回 succeeded, 失败就是 failed.  

```
- name: Create a register to represent the status if the /dev/sda6 exsited
  shell: df -h | grep sda6
  register: dev_sda6_result
  ignore_errors: True
  tags: docker

- name: Copy docker-thinpool.sh to all hosts
  copy: src=docker-thinpool.sh dest=/usr/bin/docker-thinpool mode=0755
  when: dev_sda6_result is succeeded
  tags: docker
```



注意： 
1、register变量的命名不能用 -中横线，比如dev-sda6_result，则会被解析成sda6_result，dev会被丢掉，所以不要用-    
2、[ignore_errors]()这个关键字很重要，一定要配合设置成True，否则如果命令执行不成功，即 echo $?不为0，则在其语句后面的ansible语句不会被执行，导致程序中止  



##### ansible lineinfile  



###### 追加到末尾

```yaml
ansible all -i jpuyy-com-lan -m lineinfile -a "dest=/etc/hosts line='115.239.28.20 api.jpuyy.com'" -s
```

###### 删除某一行

```yaml
ansible all -i jpuyy-com-lan -m lineinfile -a "dest=/etc/hosts line='115.239.28.20 api.jpuyy.com' state=absent" -s 
```

###### 更改某一行

```yaml
ansible all -i jpuyy-com-lan -m lineinfile -a "dest=/etc/env.yaml line='idc: sh' regexp='idc: qcloud'" -s
```





