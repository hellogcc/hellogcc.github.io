---
title: "为什么gcc在64位Solaris上编译出来的程序默认是32位的？"
date: "2014-09-14"
categories: 
  - "gcc"
---

最近发现一个问题，gcc在64位Solaris上编译出来的程序默认是32位的，而在64位Linux上编译出来的程序默认就是64位的，觉得有点奇怪，就在stackoverflow上问了一下（[http://stackoverflow.com/questions/25560539/how-does-gcc-determine-if-to-generate-a-32-bit-or-64-bit-executable-file-by-defa](http://stackoverflow.com/questions/25560539/how-does-gcc-determine-if-to-generate-a-32-bit-or-64-bit-executable-file-by-defa)）。其中一个回答给出了一个链接（[https://gcc.gnu.org/bugzilla/show\_bug.cgi?id=58833](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=58833)），原来这是Solaris有意而为之。总结一下，有以下几点：

（1）64位的gcc或程序不一定比32位运行快；

（2）Studio程序默认是32位的，gcc最好和它行为保持一致；

（3）从用户体验出发，以前都是默认生成32位程序，现在一下变成64位，用户可能需要改很多配置；

（4）64位Solaris位的gcc可以既编译32位，又编译64位，看用户自己的选择了。
