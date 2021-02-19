原始博客链接：

转换时间：
2021-01-31

注意：由于2011年HelloGCC博客遭遇了一次VPS掉电丢失，所有2011年之前（含）的博客都是从 Google Reader 等RSS中恢复的。损失了大部分的内容，非常的遗憾。如果你的电脑中保存过HelloGCC2012年之前的报告，欢迎发送给我们，非常感谢。请转发给 hellogcc.workgroup@gmail.com

# Debugger Not In Depth: Inferior Call
Posted on 2011/06/28 by yao

﻿ 1 inferior call

1 什么是inferiro call

在解释inferior call的实现原理之前，我们先解释一下inferior call是什么。

inferior call 可以被简单地理解为“调试器在当前的堆栈上，主动调用被调试程序的函数，并且正确的得到 函数的返回值（这一句好像有点多余）”。

inferior call理解起来，远比我们想象的简单，这个功能在我们的 调试过程经常会被用到。比如有这样一段代码片段，

```
extern int bar (int); void foo () { /* i is avaliable before. */ o = bar (i); }
```

假设现在程序执行到foo中，即将调用bar，我们想检查给定一个输入，函数的bar的返回值是否正确。通常这个时候我们想在GDB这里调用一下目标代码中的函数，

```
 (gdb) p bar (5) $1 = 4
```

 其实`bar (5)`是在目标端运行的，这就是一个`inferior call`。 在有些程序中，为了方便调试，开发人员会开发一些调试函数，留在程序中。这样在调试阶段，就可以利用`inferior call`这个功能，把一些程序的内部状态给打印出来。比如在调试GCC rtl的时候，我们经常在GDB中用 `p print_rtx (rtx)`，来打印rtx 的内容。这就是一个inferior call。 2 […]
