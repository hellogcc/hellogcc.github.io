转换时间： 2021-02-01

# OSDT Weekly 2020-05-20 第046期

大家5·20快乐❤️

### 近期线下活动

下一次聚会计划是V8主题。目前正在征集分享的话题。计划是线上。

同时我们欢迎场地支持和赞助。欢迎有志同道合的小伙伴联系我们。

## 编译社区的八卦信息

### GCC

- 华为的 patch，看起来不太专业，居然不带 testcase
  [PATCH] aarch64: Change the definition of Pmode [pr95182]
  https://gcc.gnu.org/pipermail/gcc-patches/2020-May/545906.html

- [PATCH] x86: Update Intel processor detection
  https://gcc.gnu.org/pipermail/gcc-patches/2020-May/545925.html

- Martin 贡献 ChangLog 自动生成的脚本
  New mklog script
  https://gcc.gnu.org/pipermail/gcc-patches/2020-May/545792.html

- dejagnu version update?
  https://gcc.gnu.org/pipermail/gcc/2020-May/232427.html

- [COMMITTED 0/2][BPF] Introduce -mxbpf and first extension
  https://gcc.gnu.org/pipermail/gcc-patches/2020-May/546032.html

- [PATCH] Implement no_stack_protect attribute.
  https://gcc.gnu.org/pipermail/gcc-patches/2020-May/545916.html

- [PATCH 0/5] rs6000: Fixes for Future, mostly testsuite
  https://gcc.gnu.org/pipermail/gcc-patches/2020-May/545843.html

### Binutils/GDB

- [PATCH 0/8] OpenRISC BFD fixups for Glibc
  https://gcc.gnu.org/pipermail/binutils/2020-May/111071.html

- PowerPC POWER10 updates to dcbf, sync and wait instructions
  https://gcc.gnu.org/pipermail/binutils/2020-May/111101.html

- Nelson Chu 的 RISC-V CSR 支持 patch set，push 了快两个月了
  [PATCH v2 0/9] RISC-V: Support version controlling for ISA standard extensions and CSR
  https://gcc.gnu.org/pipermail/binutils/2020-May/111162.html

### GLIBC

- [PATCH 00/13] Signal and error list refactoring
  https://gcc.gnu.org/pipermail/libc-alpha/2020-May/114102.html

- [PATCH 00/19] Signal mask for timer helper thread
  https://gcc.gnu.org/pipermail/libc-alpha/2020-May/114065.html

- Florian Weimer 关于动态链接库的一系列小优化
  https://gcc.gnu.org/pipermail/libc-alpha/2020-May/author.html#114099

### LLVM/Clang/LLDB/LLD

本节内容来自 [LLVM Weekly 第333期](http://llvmweekly.org/issue/333)，
[Alex Bradbury](https://www.linkedin.com/in/alex-bradbury/) 大哥持续稳定输出。

* Rust 1.0 发布五周年（当然Rust诞生的时间要早得多，到1.0花了挺久）。有篇博客介绍了这五年。
https://blog.rust-lang.org/2020/05/15/five-years-of-rust.html

* Egor Bogatov [写了篇博客讨论如何在LLVM中实现一个新的窥孔优化](https://egorbo.com/opt-for-llvm-guide.html).

* MLIR newsletter 第七期出来了。
https://llvm.discourse.group/t/mlir-news-7th-edition-5-15-2020/1015

* [ORC JIT Weekly 16](http://lists.llvm.org/pipermail/llvm-dev/2020-May/141635.html)
provides information on JITLink ELF/x86 and updates on removable code.

* Constant pools are now handled in the RISC-V load/store peephole
optimisation, improving codegen especially for loading FP constants.
[969e703](https://reviews.llvm.org/rG969e7034275).

* LLVM libc gained implementations of fabs and fabsf.
[4a39a33](https://reviews.llvm.org/rG4a39a33d44f).
这个项目可能说不定以后会发挥有意思的作用。

### RISC-V in China

最近台湾的晶芯科和上海的赛舫都开始开设线上技术直播了。讲师们的宣传照一个比一个帅。

果然现在连画个电路板都需要看颜值了么（我等相貌平庸之辈路在何方……

### TVM 社区

风平浪静。

### Mozilla

风平浪静。

### 方舟开源编译器社区

OpenArkCompiler Weekly - #11 May 17th 2020

社区动态：
1、方舟编译器社区在5月12日上午举行了会议。会议纪要如下：
小乖他爹版：https://zhuanlan.zhihu.com/p/139890748
社区版：https://gitee.com/harmonyos/OpenArkCompiler/issues/I1H1SO
2、方舟编译器社区发布会议通知，将于2020-05-19 09:00-11:00((UTC+08:00)Beijing)召开会议，会议采用Zoom，会议ID：148 489 624。
3、PLCT实验室发布了《PLCT开源进展·第一期·2020年05月16日》，将以往的周刊改为半月刊，其中介绍了PLCT实验室在各个编译方向的开源工作，其中包含了方舟编译器社区的相关工作：
https://zhuanlan.zhihu.com/p/141463489

Commits:
1、为测试结果添加XML输出格式：https://gitee.com/harmonyos/OpenArkCompiler/commit/8d3e6b2789bebf9e4962d15b6f74074d4a45ea5a
2、do inline again after me phases：
https://gitee.com/harmonyos/OpenArkCompiler/commit/dccac75f7ad10634e605c94e2dca679f928bbee2
3、为ConstvalNode添加ARM32支持：
https://gitee.com/harmonyos/OpenArkCompiler/commit/f91af1890939bb591c84912e3d1202f2322c3eb7

（本节内容来自中科院软件所智能软件研究中心PLCT实验室史宁宁的方舟周报）

### 其它社区的 Weekly

欢迎感兴趣的小伙伴提 Pull Requests 完善内容❤️
我们希望能够尽可能多的扩展下OSDT的覆盖范围。

This Week in Rust #339:
https://this-week-in-rust.org/blog/2020/05/19/this-week-in-rust-339/

Golang Weekly #312:
https://golangweekly.com/issues/312

WebAssembly Weekly #118:
https://wasmweekly.news/issue-118/

最近更新的都还挺规律 ：）

### Academic

听说PLDI可以线上免费参与了。真好啊。

### 本周工具链岗位

远程实习机会： [PLCT实验室招聘编译器/虚拟机/IDE开发实习生](https://mp.weixin.qq.com/s/bVaNK2kVGstnZ6Onkc98zQ)

## 本周推荐阅读

本周推荐：《狂热分子》

作为IT从业者，最近朋友圈完全被美国VS华为刷屏。我们不仅正在经历和见证百年一遇的大流行病（pandemic），同时也在见证人类历史上的灯塔们时不时就来那么一下子的礼崩乐坏。仿佛一夜之间所有的文明国家都开始斯文扫地，开始撒泼起来。

不是的。一向如此。只是我们看的书不够多。

这本《狂热分子》是如此的经典，自20世纪上半叶问世以来就风靡全世界；又是如此的敏感和不好评论，以至于可能即使是摘录书中的一两个段落都会让我们这个不起眼的技术公众号从微信里消失。码头哲学家埃里克·霍弗总结出的结论是普遍适用于所有人类政治结构的。埃里克·霍弗总结的是人类的运动规律。

买一本纸质书，拿在手里，小心的阅读，尽量不要让发自内心的惊叹脱口而出。

作者:  [美] 埃里克·霍弗
出版社: 广西师范大学出版社
出品方: 理想国
副标题: 群众XX圣经
原作名: True Believer : Thoughts on the Nature of Mass Movements
译者: 梁永安
出版年: 2011-6
页数: 207
定价: 34.00元
装帧: 精装
丛书: 理想国 人文精选
ISBN: 9787563374625

https://book.douban.com/subject/3057556/
