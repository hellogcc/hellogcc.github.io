---
title: "IRC聊天随笔"
date: "2011-11-25"
categories: 
  - "其它技术"
---

1.查看eh\_frame的内容。

$ readelf -wF foo

2.gdb里查看段信息。

(gdb) info files

3.记录应用程序执行的PC值

qemu-i386 -d in\_asm foo
