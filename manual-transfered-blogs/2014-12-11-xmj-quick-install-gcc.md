转换时间： 2021-02-01

# 一键编译安装gcc
Posted on 2014/12/11 by nanxiao

今天看到一篇文章 http://joelinoff.com/blog/?p=1604 ，作者通过一个shell脚本和一个Makefile，可以自动下载需要的所有安装包并且编译gcc：
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

P.S.：
（1）编译libiconv时可能会有“'gets' undeclared here“错误，请参考这篇文章。
（2）如果机器是64位的，但是缺少32位的库文件。这样在编译gcc时会出现错误：“configure: error: I suspect your system does not have 32-bit developement libraries (libc and headers). If you have them, rerun configure with –enable-multilib. If you do not have them, and want to build a 64-bit-only compiler, rerun configure with –disable-multilib.”。提示需要配置“--disable-multilib”。
