---
title: "由GFDL和GPL引出的问题"
date: "2011-06-18"
categories: 
  - "杂谈"
---

由GFDL和GPL引出的问题

注：本文根据http://gcc.gnu.org/ml/gcc/2010-06/msg00058.html整理翻译，仅供参考。如有错误或不恰当的地方，欢迎指出。

GCC的代码是GPL的，GCC的文档是GFDL的。

现在，GCC开发者希望将target hooks（目标机钩子函数）统一在一个文件（target.def）中定义，然后通过工具（genhooks）自动生成头文件，初始化和文档。这就引出了一个问题，是否可以从GPL的代码来生成GFDL的文档。相反的，对于在tm.texi中已有的文档，是否可以直接拷贝到GCC的代码中？

Mark Mitchell特意跟GCC指导委员会（Steering Committee）和RMS（Richard M. Stallman）讨论了第一个问题。问题如下，

“Can we take comments (not code) from FSF-owned GPL’d code and process  
them in some way that results in them being included in a GFDL’d manual?”

RMS的回答如下，

“If Texinfo text is included the .h files specifically to be copied into  
a manual, it is ok to for you copy that text into a manual and release  
the manual under the GFDL.”

其中，这里的“you“是指”the GCC maintainers”，并且这个许可只被限定为要将所作的改动贡献给FSF。而对于其他人，则不可以。RMS承认这是一个问题，不过他说我们需要一个GPL许可例外（GPL license exception），使其成为一个完全通用的许可，但是这需要很长一段时间。

于是，现在GCC的文档中，又多了一个tm.texi.in。对于GCC维护者，他们可以通过工具（genhooks）根据GPL的target.def和GFDL的tm.texi.in来生成GFDL的tm.texi。对于tm.texi中已有的文档，现在还无法直接拿到target.def中使用，所以只能在tm.texi.in中使用。对于新的target hooks文档，则在target.def中定义，然后自动生成到tm.texi中。

GCC的Makefile也做出了相应的修改，来生成tm.texi，并与现有的tm.texi进行比较。如果发现不一致，并且tm.texi要比tm.texi.in新，则会报如下错，

“You should edit $(srcdir)/doc/tm.texi.in rather than $(srcdir)/doc/tm.texi.”

如果发现不一致，并且tm.texi不比tm.texi.in新，则会报如下错，

“Verify that you have permission to grant a GFDL license for all  
new text in tm.texi, then copy it to $(srcdir)/doc/tm.texi.”
