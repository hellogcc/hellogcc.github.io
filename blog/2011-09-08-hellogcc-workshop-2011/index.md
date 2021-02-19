---
title: "HelloGCC WorkShop 2011"
date: "2011-09-08"
categories: 
  - "社区活动"
---

**报名网址：[http://linux.chinaunix.net/hellogcc2011](http://linux.chinaunix.net/hellogcc2011 "http://linux.chinaunix.net/hellogcc2011")**

2011年9月24日（周六）下午

【演讲主题】

1、Introduction to GCC Backend

演讲者：刘佳

拿一个简单而具体的例子介绍了GCC的工作流程，尤其GCC后端的工作流程。主要介绍了gcc是怎么处理rtl模版从而生成代码的。最后通过LLVM的后端对比一下异同。

2、GNU Tools for ARM Embedded Processors

演讲者：叶锦云

简介： 作为维护和改进GCC上ARM架构的工作的一部分，ARM将维护一个GCC工具链的分支，特别针对嵌入式内核，如ARM Cortex-R/Cortex-M系列。此外，ARM将定期的从这个分支上构建、测试并发布二进制包。发布的包可以任意的整合到工具链中，或直接使用。这个话题将主要介绍ARM建议的工作模式和计划改进GCC的要点。您将了解到更多关于GCC在嵌入式方面的应用及挑战。并期待听到您独特的见解。

3、多核时代更快断点 — Displaced stepping以及对Thumb-2指令集的实现

演讲者：齐尧

简介： 多核处理器逐渐成为主流，一些传统的调试技术无法适应新的编程方式（比如多线程编程）。如何实现一种对多线程程序更加快速的断点机制进入的调试器开发人员的视野，而displaced stepping也就应运而生。本文介绍了displaced stepping的工作原理和实现方式。结合ARM Thumb-2指令集，讲述了如何为一种新的指令支持displaced stepping。同时还介绍了基于displaced stepping的GDB non-stop工作模式。最后，会对今后的多线程调试或者多核处理器调试做一个展望。本文会帮助读者理解displaced stepping的机制和移植工作，也为读者从GDB的内部剖析了non-stop工作模式。

4、TCG与LLVM生成二进制代码性能分析

演讲者：徐国伟

简介： 现在很多模拟器采用了LLVM作为二进制翻译的后端，相对于解释执行的模式，得到了巨大的性能提升，而且由于LLVM的多平台性，通用性可以做的很好。本文基于Skyeye和Qemu两种模拟器，给出了Benchmark程序在用户态模拟下的TCG和LLVM生成的宿主机代码与x86本地编译的代码性能对比。

5、走进GCC插件时代

演讲者：邢明杰

简介：GCC从4.5开始支持插件，使得开发者可以使用插件技术来扩展编译器功能，一时间也出现了一些第三方插件。插件技术的引入，是否意味着GCC又进入了一个新的时代，它又会带来哪些问题？本话题介绍了GCC 插件技术的原理，实现，以及现有一些第三方插件；同时，也和大家分享一些插件技术背后的故事。
