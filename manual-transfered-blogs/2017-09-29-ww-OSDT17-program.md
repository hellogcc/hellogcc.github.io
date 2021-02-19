转换时间： 2021-02-01

# 这些演讲厉害了！OSDT17议程介绍
Posted on 2017/09/29 by ww	

开源开发工具大会（OSDT）将于10月21日下午在清华大学举办。今年的OSDT我们很荣幸邀请到了Go语言活跃贡献者史斌、ARM首席工程师刘江宁、TYA / HardenedLinux 低调白帽王翔、以及HelloGCC元老邢明杰（渔夫）为大家带来四场干货满满的技术报告。演讲主题涵盖量子计算与编程、RISC-V 移植 Coreboot、支持GPU的ARM快速模型子系统验证技术、Go语言编译器等。

感谢北京迪捷数原科技有限公司赞助活动场地和演讲者晚餐` (^_^)`

我们将在十一长假之后通过微信公众号进行参会报名，请感兴趣的童鞋请关注HelloGCC微信公众号，不会错过后续报名通知。
公众号搜索：hellogcc2007

关注HelloGCC微信公众号

大会演讲题目及简介如下：

1. Go语言及其编译器介绍

演讲嘉宾：史斌（Golang社区）

内容简介：

Go语言自诞生之日起就备受关注。经过几年的发展壮大，如今已经有了稳定的用户群体和社区支持，并在多种生产环境场景中得到广泛应用。史斌是Go语言社区的活跃代码贡献者，为Go语言编译器贡献了多个patches。在这次报告中他将首先带我们快速熟悉Go语言的核心概念；在此基础上对Go语言编译器的架构实现进行介绍；最后介绍他在Go编译器上的近期工作。

2. 支持GPU的快速模型子系统验证技术

演讲嘉宾：刘江宁（ARM）

内容简介：

ARM快速模型开发工具（Fast Model）是用于构建SoC上各种IP集成的系统的软件。它允许客户在系统各部件的标准确定后，马上把全栈软件跑起来。这次我将以实现GPU模型的系统方法为例，介绍如何使整个安卓系统软件栈在一个CPU和GPU集成的模拟环境下运行，最终达到验证GPU IP的目的。

Fast Model 采用独特的GPU加速技术，能在安卓上运行安兔兔这样的图形Benchmark，而安卓OS运行在用 Fast Model 的子系统模型上。这在过去几乎不可能，3D渲染模拟GPU验证慢得要死。在我们的模型上验证过的全栈软件无需任何修改就能够在硬件流片后立刻运行起来，缩短工业界开发SoC的周期。

此外，ARM提供免费的已经构建好的虚拟平台，在此平台上可以运行符合Linaro标准的Linux和Android操作系统，而这种操作系统已经被适配到ARMv8-A架构上，支持v8A的各种先进特性。

3. 固件自由RISC-V架构下的coreboot

演讲嘉宾：王翔（腾御安，HardenedLinux社区）

内容简介：

王翔平时的工作专注于固件的研究。在此次演讲中，他将首先向大家介绍Coreboot项目的由来和使用场景；然后对最近发展得如火如荼的RISC-V的特点进行分析；最后，介绍Coreboot移植到基于RISC-V的HiFive1系统过程中遇到的坑和人生感悟。

对固件安全、Coreboot以及RISC-V感兴趣的朋友一定不要错过这次跟王翔童鞋面基的机会 🙂

4. 浅谈量子计算与编程

演讲嘉宾：邢明杰（计算所、HelloGCC社区）

内容简介：

今年是量子计算机的爆发年，你或许已经听过量子计算机、谷歌的量子霸权计划、IBM和微软的量子计算机的原理，但是你肯定没有自己写过量子计算机程序 😛

邢明杰此次带来的量子计算与编程演讲可能是目前为止你在国内可以听到的唯一一场介绍量子计算机编程的入门讲座，并且，没！错！的！你有很大的概率能够听得懂哦 (￣∇￣)

5. 闪电演讲环节

如果时间充裕我们提供五分钟左右闪电演讲机会，演讲者和演讲内容待定。

我们将在十一长假之后通过微信公众号进行参会报名，请感兴趣的童鞋请关注HelloGCC微信公众号，不会错过后续报名通知。