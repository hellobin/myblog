+++
author = ""
comments = true
date = "2012-02-16T22:29:34+08:00"
draft = false
image = ""
menu = ""
share = true
slug = "windows下为有道添加星际译王词典"
tags = ["windows"]
title = "windows下为有道添加星际译王词典"

+++

&emsp;&emsp;windows下一直使用有道词典，缺陷没有离线词库。今天终于找到让有道词典可以支持离线的方法，那就是给其添加大名鼎鼎的星际译王的词典（其实星际译王也有windows版的）。    

 1.  到这里[下载](http://abloz.com/huzheng/stardict-dic/zh_CN/)自己需要的词典 

 2.  下载的文件默认是tar.bz2格式，一种linux的文件打包压缩格式（因为星际诞生于gnu/linux平台）。有道并不支持这种压缩的格式，需要解压。用winrar，等很多解压缩软件都可以。

 3.  解压出来，后缀为dz,idx,ifo三个文件。建议将其全部拷贝到X:/Program Files/Youdao/DeskDict2/localdicts  （X为你的有道安装目录）。

 4. 四 打开有道桌面词典，按照下面的顺序操作：选项…->词典管理->添加，找到刚刚解压缩获得的文件夹里的ifo格式的文件，确认后添加

&emsp;&emsp;现在可以方便的使用有道词典离线查词啦

