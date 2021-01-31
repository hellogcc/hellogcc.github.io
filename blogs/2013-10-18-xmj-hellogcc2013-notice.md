转换时间： 2021-01-31

# HelloGCC 2013技术讨论会活动通知
Posted on 2013/10/18 by xmj

演讲资料参见:https://github.com/hellogcc/HelloGCC2013
活动报名页面 http://www.hellogcc.org/?p=33732
HelloGCC 2013技术讨论会活动照片记录 http://www.hellogcc.org/?p=33562
HelloGCC 2013技术讨论会录像 http://www.hellogcc.org/?p=33732

媒体报道：开源工具链“大牛”的经验分享：HelloGCC 2013精彩演讲回顾

日程安排：
9:00 – 9:30 入场、签到
9:30 – 9:40 嘉宾介绍、致谢
9:40 – 10:30 话题：在Cling上实现空指针解引用检测机制，丁保增
10:30 – 11:20 话题：Build An Optimized C Runtime for Embedded Linux，黃敬群
11:20 – 11:30 抽奖

午餐、休息
13:30 – 14:20 话题：GCC上归纳变量的优化，程斌
14:20 – 15:10 话题：Port GCC to a new architecture – Case study: nds32，Chung-Ju Wu
15:10 – 15:40 自由讨论
15:40 – 16:30 话题：The Theory, History and Future of System Linkers，Luba Tang
16:30 – 17:20 话题：The implementation of AArch64 Neon™ in LLVM，刘江宁
17:20 – 17:30 抽奖

题 目：在Cling上实现空指针解引用检测机制
演讲者：丁保增
中科院软件所博士在读，研究方向：系统安全，虚拟化安全。在GSoC2013项目中，实现了在Cling中动态地检测空指针解引用错误。
简 介：Cling是欧洲核子研究中心(CERN)开发的C++交互式编译器。在报告中将介绍：C++交互式编译器Cling的应用场景，实现以及如何在Cling中实现动态地检测空指针解引用错误。

题 目：Build An Optimized C Runtime for Embedded Linux
演讲者：黃敬群
慣用網路暱稱是 “jserv”，長期投入開源軟件開發工作，並致力於軟硬件系統整合，同時擔任台灣聯發科技和工業技術研究院等單位的顧問，協助銜接開放系統的豐富資源和活躍的社區。近期除了投入於醫療電子產業之外，也在台灣的大學院校授課，與學生一同探討嵌入式系統和操作系統一類的議題。
http://about.me/jserv
简 介：olibc is derived from bionic libc used in Android, which was initially derived from NetBSD libc. olibc is expected to merge the enhancements done by several SoC vendors and partners, such as Qualcomm, TI, Linaro, etc., which is known to be the major difference from glibc, uclibc, and other traditional C library implementations. Typically, the code size of olibc runtime should be highly customizable. For ARM target, olibc would benefit from ARMv7 specific features like NEON, Thumb-2, VFPv3/VFPv4, and latest compiler optimization techniques. Also, olibc is released under BSD License.

题 目：GCC上归纳变量的优化
演讲者：程斌
ARM工程师，开源工具链爱好者, GCC、Newlib开发者，目前在ARM从事GNU工具链开发方面的工作。
简 介：归纳变量是循环优化中非常重要的因素。不同的IV选取方法往往导致生成代码的大小和性能有很大的差距。ARM工程师程斌在GCC的IV Optimization上做了很多的调优和修复，并在目标机上取得了明显的性能提升。他将会介绍IVO的概况和他遇到的一些有趣的问题。

题 目：Port GCC to a new architecture – Case study: nds32
演讲者：Chung-Ju Wu
Chung-Ju Wu is a compiler engineer at Andes Technology Corporation. He is currently working with other engineers on developing a complete toolchain/compiler for supporting Andes nds32 architecture.
He received M.S. degree in Computer Science from National Tsing-Hua University,
HsinChu, where he is also working towards the Ph.D degree. His research interests include GCC/Open64 compiler porting, compiler optimizations, and system software for embedded SoC integrations.
简 介：GCC, GNU Compiler Collection, has been ported to a wide variety of processor architectures. Although there are some documentation describing GCC internal infrastructures and porting mechanism, there is still gap between modifying existing ports and adding new ports in GCC.
In this session, an architecture — nds32 — is taken as case study, we will go through necessary steps of creating a new target port for GCC from scratch. This presentation starts with introducing the source files that are required to be modified and prepared. Then some fundamental naming patterns in machine description will be mentioned. We will also talk about how to improve code generation by utilizing target hooks and refine machine description patterns. Hopefully this tutorial is able to provide some guidelines for ones who are interested in porting GCC for a new architecture.

题 目：The Theory, History and Future of System Linkers
演讲者：Luba Tang
Taiwan Evolution Software Technology, Founder. Luba Tang received his M.S. degree in computer science from the National Tsing-Hua University, Taiwan. He has been a Ph.D. student in computer science department of National Tsing-Hua University, Taiwan since 2007. At the same time, he has been working in the compiler groups at Marvell, Inc. and MediaTek, Inc. since 2010. His research interests include both electronic system level (ESL) design and compilers. He had focused on iterative compiler, ahead-of-time compiler, and link-time optimization. His most recent work focus is on cloud compilation. He was the chief programmer of Starfish DSP simulator, the original author of Marvell iterative compiler, and also the software architect of MCLinker. His most recent position is the founder of Taiwan Evolution Software Technology Inc..
简 介：The tedious and tough details of linking process limit the development of system linkers for more than 30 years. Most works stopped at providing portable infrastructures; Some works ever focused on optimizing linker, but all of them become limited binary rewriter, not real and practical linkers; Only few works ever addressed on modeling linking process.
In this presentation, I will review the history of development of system linkers. I will start from introduction to 1992 OM system – the first work attempts to build an optimizing linker. I will also introduce several famous successors – ATOM and Intel PIN projects, some competitors – 1999 Alto, 2000 ICFG and 2003 Diablo link-time optimizer and eventually introduce modern optimizing linkers – Google gold, Apple ld64 and MCLinker.
Modern optimizing linkers have different models of linking. Every models have its unique strength and weakness. I will introduce the differences, advantages and drawbacks of these linkers. This part will touch ATOM model of ld64 and Fragment-reference model of MCLinker.
Finally, I will introduce a new open source project – bold. The goal of the bold project is to provide a parallel and optimizing linking infrastructure. It’s my introspection of linking models.

题 目：The implementation of AArch64 Neon™ in LLVM
演讲者：刘江宁
现任ARM首席软件工程师(Principal software engineer)， 长期从事编译器软件的设计和开发工作。从2000年至2010年任职于英特尔，期间曾在英特尔编译器实验室参与和领导开发多个编译器项目，其中包括UEFI中间代码编译器，英特尔编译器x86-64后端实现，以及动态二进制代码翻译性能优化等等。自2011年起任职ARM软件工具链部门，参与和领带开源软件的开发工作，其中包括针对ARM Cortex-M系列CPU的GCC编译器的性能优化，以及LLVM/Clang编译器针对ARM v7/v8 CPU的实现。
简 介：开发一个LLVM的新后端时，刚开始的时候是往往是比较简单直接的。但是越往后越就会发现要做好一个LLVM后端有很多要考虑的问题。ARM工程师刘江宁在LLVM后段开发时遇到一些问题。他将描述这些问题并分享解决这些问题的经验。

合作伙伴：
http://www.is.cas.cn/
20131015034226346

赞助单位：
20131015031605141
20131015031735342
20131015031933762
20131017113656792
20131015032312525
20131017113621554

媒体支持：
20131017113731822
20131017113801135
20131015032536333
20131015032633978

时间：2013年11月16日09:00 – 18:00
地点：北京市中关村南四街4号中国科学院软件研究所5号楼4层大会议厅
