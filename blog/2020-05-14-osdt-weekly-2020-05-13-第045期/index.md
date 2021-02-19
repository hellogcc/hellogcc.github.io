---
title: "OSDT Weekly 2020-05-13 第045期"
date: "2020-05-14"
categories: 
  - "杂谈"
tags: 
  - "osdt-weekly"
---

### 近期线下活动

HelloGCC&HelloLLVM 社区开始恢复聚会活动。本周末会进行QEMU讨论分享沙龙。具体的议程和讨论内容将在明后天推送，请感兴趣的小伙伴关注。第一次活动可能以线上形式进行。

后续我们将会在每周末举行一个技术主题的线上讨论。本周日（5月17日）为QEMU主题，下周日（5月24日）为V8主题，下下周（5月31日）为LLVM主题。

同时我们欢迎场地支持和赞助。欢迎有志同道合的小伙伴联系我们。

## 编译社区的八卦信息

### GCC

- GCC 10.1 Released。对比 GCC 9 的改进请移步，重点提到 C++20  
    https://gcc.gnu.org/pipermail/gcc/2020-May/232334.html
- H.J.Lu 共享的x86安全特性改进  
    \[PATCH 0/5\] Add CET host support to libcc1  
    https://gcc.gnu.org/pipermail/gcc-patches/2020-May/545459.html\[PATCH 0/3\] Add CET support to libphobos  
    https://gcc.gnu.org/pipermail/gcc-patches/2020-May/545433.html
- 大神 Ulrich Drepper 回归GNU社区了 ????  
    std::atomic\_flag::test  
    https://gcc.gnu.org/pipermail/gcc-patches/2020-May/545402.html
- 没注意到 Alexandre Oliva 已经从 redhat 换到 adacore 了，都是专注GCC的好公司  
    avoid infinite loops in rpo fre  
    https://gcc.gnu.org/pipermail/gcc-patches/2020-May/545340.html
- Richard Sandifordf 发出大批 AArch64 及 GCC 中端优化patch  
    \[PATCH\] `tree: Add vector_element_bits(_tree)` \[PR94980 1/3\]  
    https://gcc.gnu.org/pipermail/gcc-patches/2020-May/author.html#545537
- Jason Merrill 提交了大批 c++改进的 patch，应该是继续 c++20 的工作  
    \[pushed\] c++: Avoid unnecessary deprecated warnings.  
    https://gcc.gnu.org/pipermail/gcc-patches/2020-May/545547.html
- IBM在PowerPC的GCC投入上似乎得到很大的加强，Bill提交了大量Power10后端代码，部分来自上海团队。  
    \[PATCH 0/4\] rs6000: setbnc and friends \[pu\]  
    https://gcc.gnu.org/pipermail/gcc-patches/2020-May/author.html#545269

### Binutils/GDB

- \[PATCH v4\] ld: Add --export-dynamic-symbol  
    https://sourceware.org/pipermail/binutils/2020-May/110991.html
- Alan提交大量PowerPC10后端改进patch  
    Power10 byte reverse instructions  
    https://sourceware.org/pipermail/binutils/2020-May/author.html#111027
- GDB社区平静，修testsuite为主，没干货

### GLIBC

有一个重要的干货 patch set，值得仔细读读

- \[PATCH v2 00/13\] aarch64: branch protection support  
    https://sourceware.org/pipermail/libc-alpha/2020-May/113850.html

### LLVM/Clang/LLDB/LLD

本节内容来自 [LLVM Weekly 第332期](http://llvmweekly.org/issue/332)，  
[Alex Bradbury](https://www.linkedin.com/in/alex-bradbury/) 大哥持续稳定输出。

- Nikita Popov 写了一篇博客介绍如何加速LLVM构建速度：  
    https://nikic.github.io/2020/05/10/Make-LLVM-fast-again.html  
    大概是10%的速度提升。  
    （Hmm……）
- Google Summer of Code 结果出了，快来看看入选LLVM项目的学生名单吧！  
    https://summerofcode.withgoogle.com/organizations/4674300587540480/#projects  
    恭喜所有中选的学生！  
    **同时这次没有中的学生也不要灰心，欢迎来PLCT实验室实习，做LLVM！**\*  
    而且是远程实习哦：  
    [PLCT实验室招聘实习生](https://mp.weixin.qq.com/s/bVaNK2kVGstnZ6Onkc98zQ)
- Tanya Lattner 代表LLVM基金会宣布今年 LLVM 开发者大会会线上进行。  
    http://lists.llvm.org/pipermail/llvm-dev/2020-May/141436.html
- Kai Wang shared an RFC on [RISC-V vector intrinsics](http://lists.llvm.org/pipermail/llvm-dev/2020-May/141452.html).  
    紧接着，PLCT实验室号召国内RISC-V厂商积极的参与标准的制定。目前国内的厂商参与标准的制定的动作很少，可以更多一点。  
    [RISC-V Vector Extension Intrinsic RFC 开始活跃更新，我们号召国内厂商抱团参与](https://mp.weixin.qq.com/s/qAQmXwhCccVGms90lJzz2g)
- ORC JIT Weekly #15 发布。  
    http://lists.llvm.org/pipermail/llvm-dev/2020-May/141493.html
- Complex numbers, complex addition, and complex subtraction were added to the  
    standard MLIR dialect.  
    [031265a](https://reviews.llvm.org/rG031265ad8a2),  
    [5d5f61f](https://reviews.llvm.org/rG5d5f61fc894).

### RISC-V in China

赛舫科技从 SiFive China 改名为 StarFive，同时推出了「不要钱计划」，开始血洗国内IP市场。「一分钱计划」的发起方芯来科技积极迎战，宣布了「一分钱升级计划」。其他MCU企业目前都还没出来讲话。

别打了，再这么打下去，估计就要倒贴用户了????

### TVM 社区

风平浪静。

### Mozilla

风平浪静。

### 方舟开源编译器社区

上周日，PLCT实验室史宁宁主管（知乎ID：小乖他爹）按时更新了内容：

OpenArkCompiler Weekly - #10 May 10th 2020

社区动态：  
1、方舟编译器社区发布会议通知，将于2020-05-12 09:00-11:00((UTC+08:00)Beijing)召开会议，会议采用Zoom，会议ID：868 779 283。

Commits:  
1、重构maple\_me的BB接口：  
https://gitee.com/harmonyos/OpenArkCompiler/commit/302da848a614692d88ad6940ff8201f3d878fed3  
2、添加loop unrolling代码：  
https://gitee.com/harmonyos/OpenArkCompiler/commit/ec7085b2d73461a848bebc2d1e99e69619fcf3b9

### 其它社区的 Weekly

欢迎感兴趣的小伙伴提 Pull Requests 完善内容❤️  
我们希望能够尽可能多的扩展下OSDT的覆盖范围。

This Week in Rust #337:  
https://this-week-in-rust.org/blog/2020/05/05/this-week-in-rust-337/

Golang Weekly #310:  
https://golangweekly.com/issues/310

有一个 [Rob Pike 的采访](https://golangweekly.com/link/87635/web)

WebAssembly Weekly #115:  
https://wasmweekly.news/issue-115/

还是4月17日的。要变成 Monthly 了。

### Academic

现在最大的愿望是不要再听到学术泰斗仙逝的消息了。

### 本周工具链岗位

远程实习机会： [PLCT实验室招聘编译器/虚拟机/IDE开发实习生](https://mp.weixin.qq.com/s/bVaNK2kVGstnZ6Onkc98zQ)

## 本周推荐阅读

本周推荐：《一胜九败》

作者: 柳井正  
译者: 徐静波

过去几年因为参与创业的关系，阅读了不少创业者写的自传或心得。《一胜九败》是优衣库的创始人柳井正先生写的回顾，介绍了从接手优衣库前身的家庭小店开始逐步发展壮大的过程，尤其是其中各种失败的尝试。

小编一号观察到的一个有意思的现象是美国、中国大陆、日本的创业者写的书，风格是完全不同的。美国的创业者的书会让我觉得是在准备一个20分钟的TED演讲：精心准备，有明确的脉络架构，即使是写书也要有转折和伏笔，也要有升华。日本的创业者自己写的书，则更像是两个人并排或者面对面坐在一张桌子上，读者作为入赘的女婿或家中长女要被托付上家族企业的未来，现任掌舵想到什么重要的事情就事无巨细的倾倒出来。而中国的创业者的书，绝大部分不是本人写的，是职业传记写手根据二手公众号鸡汤自己二次创作的。我走眼买过几次这样的图书，阅读发现不是创业者自己写的，非常不开心。

不管是否想要创业，柳井正先生的《一胜九败》都是值得阅读的。难能可贵的是柳井正先生将自己在做决策中的种种考虑，写了出来。这是非常珍贵的。管理学大师德鲁克先生说过，「管理者就是承担有风险的决策的」，可以说作出正确的决策并承担决策失败的风险，是管理者的基本角色定义。而决策失败了怎么办以及如何做决策，一手的资料非常少。毕竟市场环境瞬息万变，公司内政治错综复杂，即使是决策者本人可能也会依赖于当时的肠子感觉来一手神决策，有太多决策的利益权衡是只能做不能说。能够记录下载自己的决策并向读者分享出来自己的成功和失败，并尝试向读者解释决策的过程，对于小编一号而言是一本珍贵的参考资料。

最后，柳井正没有说出来的，非常非常重要的一点是，为什么失败了那么多次还可以爬起来：因为输得起。账面现金流一定是要充分的，主业现金牛是稳定的，失败的过程就是在不断寻找新的现金牛的过程。或许这对于创业者是更为要命的经验教训。
