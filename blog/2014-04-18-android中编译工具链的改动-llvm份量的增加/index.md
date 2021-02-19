---
title: "Android中编译工具链的改动----LLVM份量的增加"
date: "2014-04-18"
categories: 
  - "llvm"
tags: 
  - "android"
  - "art"
  - "llvm-2"
---

最近，Android中的编译工具链发生了改动，这个改动是Android的runtime（也可以说是VM，这两种说法在Google官方文档中也多次交互出现）改动引发的。之前Android采用的是Dalvik VM，在Android2.2之前，连JIT都没有使用，只是解释执行，所以速度很慢，从Android2.2之后加入了JIT之后，一直用了相当长的时间。直到ART（Android Runtime）的出现,ART的出现也是为了进一步提高运行速度，据早期的测试结果表明，ART上的执行速度可以比Dalvik上的执行速度快一倍。所以总的来说，不管是添加JIT支持，还是现在的ART，都是为了速度更快。

ART是在Android4.4正式出现的，就是它引起了Android中编译工具链的改动。之前Dalvik拿到.dex或者优化过的.odex文件，是使用JIT然后执行的。现在ART是直接使用LLVM去做AOT（Ahead of Time），这样的话，执行速度自然就上来了，带来的牺牲是应用的安装速度会降下来，因为AOT编译是在安装的时候做的，后续的启动和执行，都使用的是AOT之后的结果。所以等于是用一次时间牺牲，换来之后的多次时间节省。 ART目前是和Dalvik同时存在系统中的，用户也可以自己选择。在系统中它们分别以Dalvik runtime (libdvm.so) 和 ART (libart.so)这两个库的形式存在，ART的源码位置也是在和Dalvik的同级位置，直接在Android目录下有个art目录。目前art目录下的设置基本上也是参照Dalvik的形式来的，几个工具也都是类似，只是把与原来的dexopt工具给换成了dex2oat，然后引入了LLVM去做编译的工作。到这个程度，LLVM等于已经参与了Android上的所有应用的编译工作，在art出现之前，LLVM只是处理Android Renderscript中的rs文件。

ART目前所带来的缺点就是占用空间增大和首次安装时间延长，这都是由AOT导致的，目前看来是没有办法避免的。还有一个问题就是目前为了同时支持Dalvik和ART，依然采用dex格式文件作为输入，但是dex格式本身就是给Dalvik所设计的可执行格式，所以如果将来真的丢掉Dalvik的话，这个dex格式就有点不伦不类了，可能到那时候这个dex格式也终将作出改变。

参考资料：

Google关于ART的介绍： [https://source.android.com/devices/tech/dalvik/art.html](https://source.android.com/devices/tech/dalvik/art.html)

Dalvik: [http://en.wikipedia.org/wiki/Dalvik\_(software)](http://en.wikipedia.org/wiki/Dalvik_%28software%29)

AOT: [http://en.wikipedia.org/wiki/AOT\_compiler](http://en.wikipedia.org/wiki/AOT_compiler)

JIT: [http://en.wikipedia.org/wiki/Just-in-time\_compilation](http://en.wikipedia.org/wiki/Just-in-time_compilation)

CSDN一篇不错的分析ART机制的文章：[http://blog.csdn.net/androidsecurity/article/details/17462529](http://blog.csdn.net/androidsecurity/article/details/17462529)

一个关于ART的YOUTUBE视频：[http://www.youtube.com/watch?v=USgXkI-NRPo](http://www.youtube.com/watch?v=USgXkI-NRPo)

一篇介绍ART的很通俗易懂的文章：[http://www.extremetech.com/computing/170677-android-art-google-finally-moves-to-replace-dalvik-to-boost-performance-and-battery-life](http://www.extremetech.com/computing/170677-android-art-google-finally-moves-to-replace-dalvik-to-boost-performance-and-battery-life)
