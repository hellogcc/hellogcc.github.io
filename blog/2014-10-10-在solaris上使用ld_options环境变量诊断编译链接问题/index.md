---
title: "在Solaris上使用LD_OPTIONS环境变量诊断编译链接问题"
date: "2014-10-10"
categories: 
  - "其它技术"
---

最近在`Solaris`上编译一款开源软件，在最后链接阶段出了问题，导致`ld`程序`core dump`。由于没有`ld`程序源代码，导致完全没思路，没办法，只好在`stackoverflow`上求教：[http://stackoverflow.com/questions/26009192/why-the-ld-crash-in-building-libgd](http://stackoverflow.com/questions/26009192/why-the-ld-crash-in-building-libgd)。从回复中我才知道可以通过设置`LD_OPTIONS`环境变量，来了解整个链接过程。举个例子：

```
LD_OPTIONS=-Dfiles,detail
```

指定`files`会输出`ld`当前处理的文件，`detail`会提供更多的信息。

可以用“`ld -Dhelp`”命令打印每个选项的详细帮助信息。

如果想详细了解`Solaris`下程序的链接过程，可以参考这篇文档：[Linker and Libraries Guide](http://docs.oracle.com/cd/E19683-01/817-3677/817-3677.pdf)。
