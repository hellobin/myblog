+++
author = ""
comments = true
date = "2013-04-06T22:33:21+08:00"
draft = false
image = ""
menu = ""
share = true
slug = "centos下mysql5.6.10源码安装"
tags = ["linux", "mysql"]
title = "centos下mysql5.6.10源码安装"

+++

从mysql5.5起，mysql源码安装开始使用cmake了，所以首先要安装cmake
可以用命令:

```Shell
rpm -qa | grep cmake
```
查询当前系统有没有安装查cmake。

下载解压安装cmake

```Shell
#tar -zvxf cmake-2.8.3.tar.gz
#cd cmake-2.8.3
#./configure
#make
#make install 
```
可能会出现没有C++编译器的错误，默认的gcc默认没有支持c++，使用以下语句安装

```Shell
#yum -y install gcc-c++
```
执行源码编译配置脚本

```Shell
	cmake \
	-DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
	-DMYSQL_UNIX_ADDR=/usr/local/mysql/mysql.sock \
	-DDEFAULT_CHARSET=utf8 \
	-DDEFAULT_COLLATION=utf8_general_ci \
	-DWITH_MYISAM_STORAGE_ENGINE=1 \
	-DWITH_INNOBASE_STORAGE_ENGINE=1 \
	-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
	-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
	-DWITH_MEMORY_STORAGE_ENGINE=1 \
	-DWITH_READLINE=1 \
	-DENABLED_LOCAL_INFILE=1 \
	-DMYSQL_DATADIR=/db/mysql/data \
	-DMYSQL_USER=mysql \
	-DMYSQL_TCP_PORT=3306
```

如果出现以下错误：

```Shell
	cmake .
	-- MySQL 5.5.8
	-- Could NOT find Curses (missing:  CURSES_LIBRARY 	CURSES_INCLUDE_PATH)
	CMake Error at cmake/readline.cmake:82 (MESSAGE):
  	Curses library not found.  Please install appropriate package,。。。
```
执行 yum -y install ncurses-devel 命令，然后删除生成的CMakeCache.txt文件，重新执行cmake
执行 `＃make  && make install` 编译安装
在运行mysqld的时候可能会遇到类似：
	`ERROR 2002 (HY000): Can’t connect to local MySQL server through socket ‘/var/lib/mysql/mysql.sock’ (2) `
	
这样的错误，只需重新确认一下mysql.sock文件的路径，在/etc/my.cnf 文件中设施一下socket=path即可。对于mysqladmin，同样需要在/etc/my.cnf 文件中[mysqladmin]下设置socket 路径。

