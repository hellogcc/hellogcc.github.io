---
title: "使用gcc-6.1.0编译SPEC2006错误参考"
date: "2016-06-02"
categories: 
  - "gcc"
---

一堆错误，还好逐个都解决了，其中：

400.perlbench，编译出错，一堆重定义，是inline的不同实现标准造成的，需要加上一个选项-std=gnu90。gnu90和c99的对extern函数的inline实现方式不同。 416.gamess 运行出错，需要加上一个选项-funconstrained-commons，参考https://gcc.gnu.org/ml/gcc-patches/2016-03/msg00465.html，gfortran文档中没有提到这个选项。 其它还有一些是strcat，memcp之类的函数，没有显式的include相应的头文件。还有的，是不在std namespace中，代码中竟然有std::。
