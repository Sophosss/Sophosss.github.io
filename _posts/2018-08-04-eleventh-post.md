---
title: Hadoop的SSH问题
author: Sophosss
layout: post
---
通常hadoop入门文章，都会让新手从搭建入手，先是配置免密码登陆，然后执行start-all.sh，然而这部分如果没有linux基础，新手搭建过程中出问题是很容易出现疑问的。

### hadoop搭建真的需要配置免密码登陆吗？

### start-all.sh脚本是如何执行的

![执行过程](https://yudaer.oss-cn-hangzhou.aliyuncs.com/pic/13.png)

执行过程

- 首先进行环境配置，通常是一些环境变量的设置
- 执行start-dfs.sh，启动hdfs
- 执行start-yarn.sh，启动yarn

### start-dfs.sh脚本是如何执行的

- 启动namenode
  - 执行bin/hdfs getconf -namenodes，从core-site.xml中的defaultFS获取namenode启动的host（有人曾经写错defaultFS，起不来namenode的原因就在这）
  - ssh到这个机器启动sbin/hadoop-daemon.sh start namenode
- 启动datanode
  - 从etc/hadoop/slaves获得所有datanode的节点的host
  - 循环每一个host，ssh到这个host执行 sbin/hadoop-daemon.sh start datanode

### [start-yarn.sh的启动过程类似于start-dfs.sh](http://start-yarn.xn--shstart-dfs-b09qv4en1ttwf2r4nxsm6ukxn6g.sh/)

### 免密码登录的秘密

上面提到的，启动datanode的时候实际上是从启动机器上ssh到所有机器上执行启动脚本，如果不设置免密码登录，启动的过程需要输入密码，很麻烦，而且启动过程可能因此中断。

### 如何设置免密码登录？

ssh在每个用户下都会有一个.ssh文件夹，里面的id_rsa.pub表示用户公钥，id_rsa表示用户秘钥，公钥和秘钥是对应的，authorized_keys里面是值得信任的公钥。如果要登录过去的机器持有该机器该用户的用户公钥，ssh过程中通过对比公钥秘钥就可以登录了。具体的做法就是将公钥的内容发往要ssh的机器，保存在authorized_keys中。

### 如何加机器

通过启动流程可以看出，启动一个datanode仅仅需要执行sbin/hadoop-daemon.sh start datanode。

### 企业如何启动

肯定不会用start-all.sh启动，通常将启动hadoop脚本写进系统的启动脚本，一旦机器自动启动，hadoop自动启动；至于是否免密码登录，看企业自身对服务器安全规划。不过可以确定一点，hadoop搭建可以不配置免密码登录。