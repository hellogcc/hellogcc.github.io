转换时间： 2021-02-01

# 在CentOS 7上安装clang
Posted on 2015/06/08 by nanxiao

EPEL网站提供了clang的RPM安装包，所以要想在CentOS 7安装clang，首先需要安装EPEL包：
```
sudo yum install epel-release
```
接下来安装 clang:
```
sudo yum install clang
```
然后就可以使用`clang`了。
