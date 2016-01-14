+++
author = ""
comments = true
date = "2014-02-18T21:43:12+08:00"
draft = false
image = ""
menu = ""
share = true
slug = "centos搭建svn服务器"
tags = ["centos"]
title = "centos搭建svn服务器"

+++

首先安装subversion，直接：

    yum -y install subversion 

`svnserve --version` 查看 返回版本号说明安装成功

建立svn库,执行：

	svnadmin create /opt/svn/kuyustore

执行上面的命令后，自动在kuyustore下建立多个文件， 分别是conf, db,format,hooks, locks, README.txt。

配置svnserve

 进入上面生成conf目录， 有以下几个文件:

 authz, passwd, svnserve.conf

其中authz 配置用户权限，设置哪些用户可以读写哪些目录,passwd是设置用户和密码的, svnserve是主配置文件设置svn相关的操作。

passwd文件:

[users]

\# 用户名 =密码

kuyu=123456

以上表示建立了用户kuyu，注意行首不能有空格

authz文件：

[/]

kuyu= rw

。。。

意思是用户kuyu对所有的目录有读写权限

svnserve.conf文件：

	[general]

 	auth-access = write
	password-db = passwd
 	authz-db = authz
 	realm = kuyustore

启动svn服务

	svnserve -d -r /opt/svn

然后就可以通过TortoiseSVN客户端输入 svn://ip/kuyustore 链接svn服务了。虽然git等分布式版本控制工具是大势所趋，但是使用svn进行版本管理相对git来讲有以下优势

1. 相对简单，比git学习成本低。
2. 权限控制比git好，可以精确控制目录权限
3. svn支持空目录，windows平台有较好的支持
