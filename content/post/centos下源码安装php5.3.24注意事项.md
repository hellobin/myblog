+++
author = ""
comments = true
date = "2013-04-14T22:13:13+08:00"
draft = false
image = ""
menu = ""
share = true
slug = "centos下源码安装php5.3.24注意事项"
tags = ["php", "centos"]
title = "centos下源码安装php5.3.24注意事项"

+++



1.注意事项.    
 &emsp;&emsp;需要先安装libxml2 和 libxml2-devel    
2.php.ini的路径    
&emsp;&emsp;在Unix上，php.ini文件缺省放在/usr/local/lib上面，因为缺省的存放路径是/lib，但是可以在编译的时候使用--with-config-file-path参数 来修改php.ini的存放位置，例如你可以使用--with-config-file-path=/etc把它存放到/etc下面，然后可以从源码包中拷贝php.ini-develop 或者 php.ini-product到/etc/php.ini并修改使之满足需要。    
3.安装php的mcrypt扩展(动态加载编译)       
&emsp;&emsp;执行 `yum list installed|grep mcrypt `查看mcrypt及依赖库是否安装，没有的话执行 `yum -y install libmcrypt libmcrypt-devel mcrypt mhash`安装。下载php下的mcrypt扩展或者直接下载php的完整安装包,切换到php代码下的 ext/mcrypt目录执phpize命令（phpize是用来扩展php扩展模块的，通过phpize可以建立php的外挂模块）执行完后，会发现当前目录下多了一些configure文件，执行:     
   
    ./configure --with-php-config=/usr/local/bin/php-config` //根据具体的安装路径
    make && make install

最后出现提示：    
`Installing shared extensions: /path` //path可在 php-config 中的extension_dir处设置

接下来在php.ini 中修改

	extension_dir=/path
	extension=mcrypt.so



4.在安装gd扩展的时候 如果出现 `configure: error: png.h not found.`的错误

需要安装一下 libpng-devel    
`yum -y install libpng-devel` 即可



5.在安装curl扩展的时候如果出现：    

	configure: error: Please reinstall the libcurl distribution -
	easy.h should be in /include/curl/

解决办法同4，安装一下curl的devel包就ok了    

 	yum -y install curl-devel    
6.在用phpmyadmin导入数据经常会出现提示:*没有接收到要导入的数据*。可能是文件名没有提交，也可能是文件大小超出 PHP 限制。phpmyadmin在选择文件时会有最大限制的提示，对比下待导入数据的大小，然后打开php.ini,查找`upload_max_filesize`关键字，将其修改为比待导入数据稍大些的值，然后重启 web服务器即可。    