+++
author = ""
comments = true
date = "2013-04-13T22:40:08+08:00"
draft = false
image = ""
menu = ""
share = false
slug = "magento-1.7.02安装问题"
tags = ["magento","php"]
title = "magento 1.7.02安装问题"

+++

&emsp;&emsp;很明显错误的提示是当前数据库不支持InnoDB storage engine，但是通过show engines命令以及phpinfo()函数查看，mysql的InnodB都是正常的，后来搜到magento中文社区的[一篇文章](http://www.magentochina.org/blog/install-magento-on-mysql-5-6),原因是Mysql 5.6.1版本已经已经丢弃了`have_innodb`这个函数.而Magento1.7.0.2以下都是使用该函数检测Mysql是否激活了InnodB引擎。而我的数据库版本正好是5.6.10按照文章中的修改方法:    
在文件app/code/core/Mage/Install/Model/Installer/Db/Mysql4.php中找到以下代码    

	$variables  = $this->_getConnection()->fetchPairs('SHOW VARIABLES');
在其后加上:

	if (!isset($variables['have_innodb'])) 
	{
 		$engines = $this->_getConnection()->fetchPairs('SHOW ENGINES');        
 		return (isset($engines['InnoDB']) && ($engines['InnoDB'] == 'DEFAULT' || $engines['InnoDB'] == 'YES');
 		...
原来的return语句注释掉或者保留都不会影响。刷新一下即可继续正常安装了。

