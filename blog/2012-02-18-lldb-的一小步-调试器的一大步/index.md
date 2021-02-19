---
title: "lldb 的一小步 调试器的一大步"
date: "2012-02-18"
categories: 
  - "lldb"
---

今天下午无聊，想到了lldb。llvm大名鼎鼎，lldb作为一个子项目，真不知道什么情况。于是，就从svn 上checkout代码，自己试着在linux上编译了一下。很遗憾，lldb对linux的支持很有限，没有编译通过（这个不意外，因为lldb主要支持mac os）。正准备放弃，想着，虽然编译不过，也可以看看代码吧。结果，代码让我感觉到了惊喜，甚至是希望。

lldb的代码结构有些类似java项目的代码结构，利用目录，把源代码分开。每个文件功能单一。这样的形式，我十分喜欢。第一次看到，就有豁然开朗的感觉。和GDB形成鲜明对比，所有的文件在一个目录下边，每个文件都包含了若干个功能，模块的边界不是很清晰。过了一遍lldb的代码，虽然觉得功能远不如GDB强大，但是我觉得，理想的调式器，代码结构就应该是这样的。

这里我并不对lldb的目录结构做什么详细的讲解（我也不懂），就是想通过这个目录结果，展示lldb是一个多么结构清晰的调式器。source目录里边有若干个目录Breakpoints, Commands, Interpreter, Host, Expression, Target, Symbol, Plugins。除了Plugins 目录，我想所有人看到，都能猜想到每个目录里边文件的作用吧，多清楚！

下来再看看Plugins里边的目录吧。这个目录里边的内容，我猜想都是应该按照plugin的形式，存在于lldb的。看了以后，便觉得的确是这样。Plugins目录下边，还有很多子目录，比如ABI，DynamicLoader，ObjectFile，OperatingSystem，UnwindAssembly，等等。每个目录里边，都有这个子目录的具体实现，比如DynamicLoader目录里边就有Darwin-Kernel MacOSX-DYLD POSIX-DYLD Static 四种不同loader的实现。可以想象，lldb可以选择性的，插入一种dynamic loader的实现。

再看看Plugins/UnwindAssembly 目录里边，有每个体系结构，自己按照指令进行unwind。GDB里边也有类似的功能，不过在GDB中是用 prologue analyzer 的hook函数来实现的。lldb里边，每个子类实现UnwindAssembly，这样就有了自己的unwinder。实现方式更加OO。

再看看Plugins/SymbolFile/目录里边，看目录名字就知道是关于symbol的，果不其然，有两个目录 DWARF和Symtab。大家都能猜到，每个目录是实现读取不同的调试信息格式的。

有了上边的例子，就是给我这样的感觉，当你一层一层看目录结构的时候，代码的结构也就清晰了。我随便看了几个文件，注释很少，但是我不会觉得陌生或者无从下手。因为，这样的代码设计，已经把复杂的debugger，化整为零，便于消化和理解。相比，GDB的设计，只能用“残忍“来形容了。浏览lldb的文件，有些代码片段，总会让我想起eclipse里边经常用到的代码。lldb的设计者在最开始就考虑到了“插件化”，这是一个很好的信号。

有了这么好的设计，理解起来，就没有那么多的恐惧了。以前觉得pathdb的结构不错，现在发现，lldb要比pathdb的结构好很多。短期内，GDB的统治地位，特别是在Linux上，无法改变，但是lldb这样的结构，还是很有希望在将来和GDB竞争的。
