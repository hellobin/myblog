+++
author = ""
comments = true
date = "2014-03-08T22:52:24+08:00"
draft = false
image = ""
menu = ""
share = true
slug = "centos的逆袭"
tags = ["centos", "service"]
title = "centos的逆袭"

+++

![centos.png](http://user-image.logdown.io/user/1175/blog/1158/post/98892/NdKBG3eTS2mFhnM7n1IS_centos.png)    
&emsp;&emsp;最近想用python搭个网站，研究了目前主流服务器的配置，最后操作系统决定选择CentOs，为什么选它呢： 

1. 免费  
2. 稳定 
3. 以前用过，容易上手和维护~

&emsp;&emsp;这里简单介绍一下吧，有需求的同学可以参考    
&emsp;&emsp;CentOS的英文全称是Community Enterprise Operating System,翻译一下就是社区版企业操作系统，Linux的众多发行版之一，是由Red Hat Enterprise Linux（大名鼎鼎的小红帽）的开源代码重新修改编译而成。用时下比较流行的一个词语就是CentOS其实就是一个彻头彻尾的山寨版 RHEL，而且人家还是完全合法的山寨。众所周知Linux的内核发布遵循GNU的GPL开源协议，红帽在linux内核的基础上开发面向企业用户的linux发行版，其所有的源代码也必须开源RHEL高度的稳定性和安全性在服务器领域是有口皆碑的。所以提到CentOS不能不介绍下RHEL。      
&emsp;&emsp;RHEL（Red Hat Enterprise Linux）是由RedHat公司在linux内核的基础上开发的面向企业用户的众多linux发行版之一 ，由于其强大的安全性和高度的稳定性，在web服务器端被广泛使用。RHEL在发行的时候，有两种方式，一种是二进制的发行方式，另外一种是源代码的发行方式。无论是哪一种发行方式，你都可以免费获得（例如从网上下载），并再次发布。但如果你使用了他们的在线升级（包括补丁）或咨询服务，就必须要付费（这也是红帽的主要盈利模式）。RHEL 一直都提供源代码的发行方式，CentOS 就是将 RHEL 发行的源代码去掉RedHat商标之后重新编译一次，形成一个可使用的二进制版本。由于 LINUX 的源代码是 GNU，所以从获得 RHEL 的源代码到编译成新的二进制，都是合法的。只是 REDHAT 是商标，所以必须在新的发行版里将 REDHAT 的商标去掉。REDHAT 对这种发行版的态度是："我们其实并不反对这种发行版，真正向我们付费的用户，他们重视的并不是系统本身，而是我们所提供的商业服务。" 所以，CentOS 可以得到 RHEL 的所有功能，甚至是更好的软件。但 CentOS 并不向用户提供商业支持，当然也不负上任何商业责任。如果你要将你的 RHEL 转到 CentOS 上，因为你不希望为 RHEL 升级而付费。当然，你必须有丰富 linux 使用经验，因此 RHEL 的商业技术支持对你来说并不重要。但如果你是单纯的业务型企业，那么还是建议你选购 RHEL 软件并购买相应服务。这样可以节省你的 IT 管理费用，并可得到专业服务。一句话，选用 CentOS 还是 RHEL，取决于你所在公司是否拥有相应的技术力量。        
               ![linux-marketshare2.jpg](http://user-image.logdown.io/user/1175/blog/1158/post/98892/uz90Xz8IQX6aA9r92F1q_linux-marketshare2.jpg)    
               &emsp;&emsp;&emsp;&emsp; &emsp;&emsp;&emsp;图 linux阵营里各发行版在服务器端的市场份额
                          
&emsp;&emsp;从渊源上讲CentOS和RHEL完全是一奶同胞的亲兄弟，差别只是RHEL有红帽这个有钱的干爹。从份额上看为什么傍干爹的RHEL干不过自生自灭的CentOS，老湿窃以为有几个原因：1.RHEL虽然根正苗红，而且有正规的技术支持和商业服务，但是服务费用确实有点高哈，号称每台一年的服务费用1000刀，对于有几十上百台服务器的用户来讲这是笔不小的预算，高富帅才玩得起；2.对于服务器而言可靠性和稳定性非常重要,RHEL每五年左右更新一次，在系统的稳定性和安全性上有着极大的优势，但是对许多服务不利，比如MySQL和PHP等工具，它们在这五年的发布周期中可能会进行繁重的开发和大量的修改，尽管YUM工具的出现已经解决了软件包的更新问题，但是由于CentOS滞后于RHEL发布，这一过程中可能会修复一些最新版RHEL中的已知小bug,这也算是CentOS的一个优势；3.其它发行版本诸如fedora、Ubuntu等在作为服务器在稳定上与这二位相比还是略逊一筹; 3.CentOS 是RHEL的克隆版，如果需要可以随时平滑切换到 RHEL，从而享受红帽的服务支持。    
&emsp;&emsp;所以从创业的角度看，CentOS作为服务器无疑是首选，性价比最高，而且创业成功屌丝逆袭之后需要相应的技术支持和商业服务来降低运营风险时也可以很方便平稳的过渡到RHEL，傍上红帽这棵大树。一句话，选用 CentOS 还是 RHEL，不仅仅取决于你所在公司是否拥有相应的技术力量，还取决于一旦出了问题能不能找到个背黑锅的。    
