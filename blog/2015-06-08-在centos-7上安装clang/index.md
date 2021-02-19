---
title: "在CentOS 7上安装clang"
date: "2015-06-08"
categories: 
  - "clang"
---

[EPEL](http://fedoraproject.org/wiki/EPEL)网站提供了`clang`的`RPM`安装包，所以要想在`CentOS 7`安装`clang`，首先需要安装`EPEL`包：

```
sudo yum install epel-release
```

接下来安装 `clang`:

```
sudo yum install clang
```

然后就可以使用`clang`了。
