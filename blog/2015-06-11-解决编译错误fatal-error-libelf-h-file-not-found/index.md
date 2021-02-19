---
title: "解决编译错误“fatal error: 'libelf.h' file not found”"
date: "2015-06-11"
categories: 
  - "其它技术"
---

最近在编译一个开源项目时，遇到这个编译错误：

```
fatal error: 'libelf.h' file not found
#include <libelf.h>
     ^
1 error generated.
```

解决方法是安装`elfutils-libelf-devel`这个软件包：

```
yum install elfutils-libelf-devel
```

或：

`dnf install elfutils-libelf-devel`
