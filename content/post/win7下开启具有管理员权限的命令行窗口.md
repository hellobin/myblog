+++
author = ""
comments = true
date = "2013-01-13T22:19:53+08:00"
draft = false
image = ""
menu = ""
share = false
slug = "win7下开启具有管理员权限的命令行窗口"
tags = ["windows"]
title = "win7下开启具有管理员权限的命令行窗口"

+++

&emsp;&emsp;使用vim在win7下敲代码的时候，发现最后保存的时候提示错误 ：E212：无法打开并写入文件，很是奇怪    

&emsp;&emsp;不论是新建文件还是编辑已有的文件，问题现象一致，猜想应该是所在目录的权限设置问题。但是问题又只在

通过cmd窗口启动vim时才会出现，使用右键-->vim编辑打开文件是ok的，查了一下，原来Windows7在开启

UAC的情况下使用运行启动的cmd.exe是没有管理员权限的。而我的系统文件夹均是在图形界面下创建的，默认是

管理员，其他用户没有写的权限。    

&emsp;&emsp;那么，如何使用管理员权限启动cmd或者其他程序呢，一个简单的方法是：    

&emsp;&emsp;*在开始菜单的搜索框里输入cmd或者你要运行的程序名称，然后按下ctrl + shift + enter* 即可