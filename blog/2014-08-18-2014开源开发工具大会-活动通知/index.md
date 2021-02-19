---
title: "2014开源开发工具大会（原HelloGCC技术讨论会）"
date: "2014-08-18"
categories: 
  - "社区活动"
---

演讲资料参见：[https://github.com/hellogcc/OSDT2014](https://github.com/hellogcc/OSDT2014) [2014开源开发工具大会见闻录](http://www.hellogcc.org/?p=34042 "2014开源开发工具大会见闻录")

【大会简介】 2014开源开发工具大会（原HelloGCC技术讨论会）是由HelloGCC（www.hellogcc.org）工作组举办的年度开源技术大会。我们希望通过自由，开放，共享的方式来增进大家相互的交流。目前话题主要涉及开源工具链，开源开发工具方面。感谢演讲者为我们贡献精彩的话题 ，感谢各单位和朋友对我们的赞助和支持，欢迎大家免费报名参加。 往年活动： \*) HelloGCC Workshop 2013: [http://www.hellogcc.org/?p=33518](http://www.hellogcc.org/?p=33518 "http://www.hellogcc.org/?p=33518") \*) HelloGCC Workshop 2012: [http://linux.chinaunix.net/hellogcc2012](http://linux.chinaunix.net/hellogcc2012 "http://linux.chinaunix.net/hellogcc2012") \*) HelloGCC Workshop 2011: [http://linux.chinaunix.net/hellogcc2011](http://linux.chinaunix.net/hellogcc2011 "http://linux.chinaunix.net/hellogcc2011") \*) HelloGCC Workshop 2010: [http://linux.chinaunix.net/hellogcc2010](http://linux.chinaunix.net/hellogcc2010 "http://linux.chinaunix.net/hellogcc2010") \*) HelloGCC Workshop 2009: [https://sites.google.com/site/hellogccworkshop/hui-yi-ri-cheng](https://sites.google.com/site/hellogccworkshop/hui-yi-ri-cheng "https://sites.google.com/site/hellogccworkshop/hui-yi-ri-cheng")

【日程安排】 \[table\] ,2014年9月13日（周六）中科院软件所5号楼4层大会议厅 9:00 – 9:30, 入场、签到 9:30 – 9:40, 嘉宾介绍、致谢 9:40 – 10:30, Performance Testing for GDB，齐尧 ([video](http://www.infoq.com/cn/presentations/performance-testing-for-gdb "video")) 10:30 – 11:20, Design and Implementation of GCC Register Allocation，程皇嘉(Kito Cheng) ([video](http://www.infoq.com/cn/presentations/design-and-Implementation-of-gcc-register-allocation "video")) 11:20 – 11:30, 抽奖 ,午餐、休息 13:30 – 14:20, pyOCD -- gdbserver for ARM CMSIS-DAP，王哲宇 14:20 – 15:10, Beyond Compiler Engineering，唐文力(Luba Tang) 15:10 – 15:40, 自由讨论 15:40 – 16:00, 开源工具构建ARM JTAG调试环境，翟京(Orson Zhai) ([video](http://www.infoq.com/cn/presentations/open-source-tools-to-build-arm-jtag-debugging-environment "video")) 16:00 – 16:50, Android的全系统符号执行工具-Android\_S2E，康烁 ([video](http://www.infoq.com/cn/presentations/whole-system-symbol-Implementation-tools-android-s2e "video")) 16:50 – 17:00, HelloGCC 闪电秀，阿里平寿 (kennyluck) ([video](http://www.infoq.com/cn/presentations/the-hellogcc-show "video")) 17:00 – 17:20, Android安全 – 伪命题?，周亚金 ([video](http://www.infoq.com/cn/presentations/android-security-pseudo-proposition "video")) 17:20 – 17:40, 抽奖，合影 \[/table\]

题目：Performance Testing for GDB 演讲者：齐尧 简介： GDB development process has no standard mechanism to show the performance of GDB snapshot or release is improved or worsened. Performance regressions do show up periodically. We recently add a new performance testing framework in GDB. In this session, we introduce this performance testing framework and show how do we use it to track GDB performance during development. Then, we show writing performance test cases on top of this framework and performance problems exposed by these cases. Finally, we discuss about adding more test cases and fixing other performance problems in the future.

题目：Design and Implementation of GCC Register Allocation 演讲者：程皇嘉(Kito Cheng) 晶心科技(Andestech)工程師，目前專注於 nds32 CPU 的 GCC，進行效能及程式大小的調校。 简介：Register Allocation 在編譯器後端是個相當重要的階段，其結果將直接影響最終程式的效能以及大小，而 Register Allocation 在 GCC 中一直是個難以對付的東西，所幸近年來有 Red Hat 的高手將其改善，於 GCC 4.4 時將原本的Local/Global 兩階段 RA，轉換為 IRA，並且在 GCC 4.8 時LRA 正式上路，預計將慢慢取代 GCC 中數一數二複雜的Reload，而本報告中從基礎的 Graph Coloring-based Register Allocation 的理論介紹，並將其對應回目前GCC IRA/LRA 的實作，一探其運作原理。

题目：pyOCD -- gdbserver for ARM CMSIS-DAP 演讲者：王哲宇 简介：pyOCD是目前mbed社区为M系列MCU所提供的调试服务工具。该工具主要基于CMSIS DAP接口，为不同的M系列厂商提供了统一的调试接口。pyOCD和目前的GNU Tools for ARM Embedded Toolchain相互结合，为用户提供一个完整的M系列MCU的开发工具。此次主题从嵌入式调试技术的现状展开，介绍一些比较新颖的调试技术例如在MCU上的本地gdb stub，以及嵌入式gdb server和PC上的remote stub的一些异同。详细介绍pyOCD的内部实现和各模块的功能，同时也会简要介绍一下pyOCD目前在Launchpad上的发布情况，并鼓励开源爱好者们参与到pyOCD的贡献中来.

题目：Xposed for ART on Android 演讲者：唐文力(Luba Tang) 简介：本次 talk 會介紹 Xposed 移植到 ART 平台上所會遇到的問題與對應的解法。 Xposed 提供 Android 上一個可以跨 framework 的平台，大大縮減了 Android 系統維護的成本。 然而目前 Xposed 尚無法再最新的 Androdi ART 平台上運作。 本次 talk 會介紹 Xposed 在 Dalvik 上運作的原理，以及移植到 ART 平台上所會遇到的問題與對應的解決方案。如果時間允許的話，也會順便 demo 目前的成果。 相關資料 [http://repo.xposed.info/module-overview](http://repo.xposed.info/module-overview "http://repo.xposed.info/module-overview")

题目：开源工具构建ARM JTAG调试环境 演讲者：翟京(Orson Zhai) 简介： 采用全栈开源的方式（软硬件都开源）搭建 ARM处理器的JTAG调试环境。 开源软件 – OpenOCD， GDB，KiCAD 开源硬件 – Openboard

题目：Android的全系统符号执行工具-Android\_S2E 演讲者：康烁 简介： 1、符号执行的介绍和应用领域 2、全系统符号执行S2E 3、移植全系统符号子执行平台S2E到安卓的过程和方法 4、演示一个在Android\_S2E上符号执行Android的本地程序的例子

【大会地址】 北京市海淀区中关村南四街4号中国科学院软件研究所5号楼4层大会议厅 ![交通图](images/软件所.jpg)

【赞助单位】 \[table th="0"\] 钻石赞助商, [![Skymizer](images/Skymizer.png)](http://www.skymizer.com/) 白金赞助商, [![Xiaomi_Logo](images/Xiaomi_Logo.png)](http://www.mi.com/) , [![互动出版网](images/china-pub.jpg)](http://www.china-pub.com) , [![CSDN-CODE-logo](images/CSDN-CODE-logo.png)](http://code.csdn.net) 赞助商, [![ARM](images/20131015031605141.png)](http://www.arm.com) 媒体支持, [![InfoQ](images/20131017113801135.jpg)](http://www.infoq.com/cn/) , [![CSDN](images/20131017113731822.gif)](http://www.csdn.net) , [![弯曲评论](images/Tektalk_logo_seal.gif)](http://www.valleytalk.org) , [![ChinaUnix](images/20131015032536333.png)](http://www.chinaunix.net) 合作单位, [![http://www.is.cas.cn/](images/20131113015828421.jpg)](http://www.is.cas.cn) , [![20131015034226346](images/20131015034226346-300x300.gif)](http://www.opencas.org/) \[/table\]

【餐饮、礼品】 \[table th="0"\] 午餐, 前100名签到报名者 茶水饮料, 会场提供 [小米盒子增强版](http://www.mi.com/hezis "http://www.mi.com/hezis"), 抽奖6个 CSDN蓝牙音箱, 抽奖3个 CHINAPUB礼卷, 前150名签到报名者 CHINAPUB图书, 抽奖10本 CHINAPUB礼品, 展位领取 \[/table\]
