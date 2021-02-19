---
title: "一键编译安装gcc"
date: "2014-12-11"
categories: 
  - "gcc"
---

今天看到一篇[文章](http://joelinoff.com/blog/?p=1604)，作者通过一个`shell`脚本和一个`Makefile`，可以自动下载需要的所有安装包并且编译`gcc`：

```
$ # Download the scripts using wget.
$ mkdir /opt/gcc-4.9.2
$ cd /opt/gcc-4.9.2
$ wget http://projects.joelinoff.com/gcc-4.9.2/bld.sh
$ wget http://projects.joelinoff.com/gcc-4.9.2/Makefile
$ chmod a+x bld.sh
$ make
[output snipped]
$ # The compiler is installed in /opt/gcc-4.9.2/rtf/bin
```

我试了一下，果然很方便。只要有一台可以联网的机器就可以了。感兴趣的朋友可以试一试。

P.S.： （1）编译`libiconv`时可能会有“`'gets' undeclared here`“错误，请参考这篇[文章](http://www.cnblogs.com/hjj801006/p/3988220.html)。 （2）如果机器是`64`位的，但是缺少`32`位的库文件。这样在编译`gcc`时会出现错误：“configure: error: I suspect your system does not have 32-bit developement libraries (libc and headers). If you have them, rerun configure with --enable-multilib. If you do not have them, and want to build a 64-bit-only compiler, rerun configure with --disable-multilib.”。提示需要配置“`--disable-multilib`”。
