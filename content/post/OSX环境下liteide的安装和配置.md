+++
author = ""
comments = true
date = "2015-04-26T21:56:59+08:00"
draft = false
image = ""
menu = ""
share = true
slug = "OSX环境下liteide的安装和配置"
tags = ["golang", "OSX"]
title = "OSX环境下liteide的安装和配置"

+++

&emsp;&emsp;首先要安装qt

	$  brew update && brew install qt
&emsp;&emsp;clone liteide代码

	$  git clone https://github.com/visualfc/liteide.git
	$  cd liteide/build
	$  ./update_pkg.sh
	$ export QTDIR=/usr/local/Cellar/qt/xxx  //qt的版本号
	$ ./build_osx.sh
	$ ./deploy_osx_qt4.sh
	
&emsp;&emsp;最后的结果在 liteide/build/liteide,但是liteide/liteidex/liteide/bin 下面也有一个更小的版本。	
&emsp;&emsp;liteide需要gdb来调试go程序，check下系统有安装了gdb。(命令行下敲gdb有版本信息输出，有安装的话
用 `which gdb` 看下gdb的路径，签名的时候用得着)。没有的话
    
	brew update && brew install gdb
	
然后还要给gdb签名，需要创建一个code signing证书。
![](http://77g3g7.com1.z0.glb.clouddn.com/gdbcert.png)
&emsp;&emsp;如图所示，一直continue，直到选择证书存放地址，选择system。右键刚才创建的 gdbCert 证书，选择Get Info，点击“信任”，
会显示可以自定义的信任选项。代码签名”选择“总是信任” （Code Signing to Always Trust）。
&emsp;&emsp;执行：

	$codesign -s gdb-cert /path/to/gdb
然后重启下系统，在liteide中打开view->edit current environment，在配置文件中设置下

	LITEIDE_GDB=/path/to/gdb
然后就可以在liteide中尽情的享受调试go的快感。。。（目前免费方案中最好用的golang ide，无障碍单步调试）