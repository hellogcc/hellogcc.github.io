---
title: "Solaris搭建64位C语言开发环境"
date: "2014-05-12"
categories: 
  - "其它技术"
---

刚来公司时，公司的C程序还是32位的。后来我阅读了一些资料，觉得64位的程序才是真正的趋势，所以就开始尝试着开发64位的程序。这篇文章介绍如何在Solaris下搭建64位C语言开发环境，希望给需要的朋友一点帮助。

（1）gcc

Solaris的/usr/sfw/bin/gcc可以用来编译64位C程序，但是需要加-m64编译选项，否则默认编译出来的是32位程序。此外也可以从gcc的[官网](http://gcc.gnu.org/)下载gcc源代码，自行编译安装，但是要注意编译出来的gcc需要是64位的。

（2）gdb

调试64位C程序需要64位的gdb，gdb的安装步骤如下（以7.6版本为例）：

1）gunzip gdb-7.6.tar.gz  
2）tar xvf gdb-7.6.tar  
3）export CC="/usr/sfw/bin/gcc -m64"  
4）mkdir build\_gdb  
5）cd build\_gdb  
6）../gdb-7.6/configure –prefix=“/…/…(a folder path)”  
7）make  
8）make install

需要注意以下两点：

a）Solaris下的tar程序不支持"-z"选项，所以只能先调用gunzip，再调用tar，不能一步搞定：tar -xzf gdb-7.6.tar.gz。

b）目前gdb的最新版本是7.7，在Solaris下编译会有错误。解决办法也很简单，可以参考[这篇文章](http://permalink.gmane.org/gmane.comp.gnu.binutils/64832)。

（3）参考资料

个人认为Oracle的这本[《Solaris 64-bit Developer’s Guide》](http://docs.oracle.com/cd/E19455-01/806-0477/806-0477.pdf)，是在Solaris下开发64位C程序最好的资料。每一位C语言开发者都应该看一下，相信都能受益匪浅。
