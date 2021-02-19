---
title: "GDB python的一些奇怪警告"
date: "2013-06-17"
categories: 
  - "gdb"
---

你如果使用从source build的gdb，那么最近就有可能碰到一些奇怪的python的问题，比如，在启动gdb的时候，

Python Exception <type 'exceptions.NameError'> name 'os' is not defined

一般这样的情况，可以在configure的时候加上  --prefix=XXX，在make以后，make install。GDB 就会到install的python dir 去找python moudle。在[这里](http://sourceware.org/ml/gdb/2012-10/msg00102.html)有个讨论。

如果你碰到了这样的警告：

Python Exception <type 'exceptions.ImportError'> No module named gdb:

假设在gdb build dir， 这样启动gdb，

./gdb -data-directory ./data-directory
