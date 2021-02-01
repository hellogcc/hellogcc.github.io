转换时间： 2021-02-01

# 解决编译错误“fatal error: ‘libelf.h’ file not found”
Posted on 2015/06/11 by nanxiao

最近在编译一个开源项目时，遇到这个编译错误：
```
fatal error: 'libelf.h' file not found
#include <libelf.h>
     ^
1 error generated.
```
解决方法是安装elfutils-libelf-devel这个软件包：
```
yum install elfutils-libelf-devel
```
或：
```
dnf install elfutils-libelf-devel

```
