---
title: "gdb测试框架dejagnu的介绍"
date: "2011-08-19"
categories: 
  - "gdb"
---

前些天有人问到如何测试交叉编译的gdb，和有如何与板子一起工作进行测试。 其实答案相对简单，就是提供一个board file，根据自己的板子和环境，进行 一些定制。但是，好像很多人对觉得测试交叉编译的gdb或者gcc都挺神秘的， 我觉得有必要写一点点来介绍一下dejagnu。

### dejagnu的测试框架

GNU toolchain都在使用dejagnu作为自己的测试框架，所有的测试用例都是用 Tcl/Expect编写的。我们不去介绍dejagnu内部是怎样做的，而是从外部来观察 如何使用dejagnu来为GNU toolchain编写测试用例。我们可以把GNU toolchain 的测试代码分为三个部分，

- 公共的基础函数。比如在gdb测试中，我们需要一个公共函数来把测试代码编译为可执行文件，或者，一个公共函数来启动gdb等等。
- 板子级别的配置文件（board file）。这些文件和具体的测试场景有关系，比如，
    - natively build的gdb测试过程中，就不需要gdbserver。cross build的gdb测试过程就需要gdbserver。
    - 不同的测试场景需要不同的host gcc，
    - 远程调试需要知道远程机器的host name和端口号，等等
- 测试用例。比如给gdb发送一个命令，通过输出来判断是否正确。在gdb的所有测试代码中，这一类占了绝大部分。

### 测试用例

测试用例一般都是针对gdb的某个功能或者某个bug来设计的一系列gdb的操作，和对应这些操作后，应该得到的输出。简单的说，就是让gdb 执行一个或者多个命令，然后从gdb的输出来判断这个命令的执行正确与否。[GDBTestcaseCookbook](http://sourceware.org/gdb/wiki/GDBTestcaseCookbook) 详细介绍了如何在写测试用例。我们在 这里也不介绍了。

### 板子级别的配置文件（board file）

board file 听起来有些玄妙，我们先看看它在什么时候需要，

- 需要一个特殊的编译器，比如非gcc的编译器，来编译测试代码，
- 需要测试一个非gdb的调试器，或者没有在安装缺省路径下的调试器，
- 需要测试一个交叉调试器，在远程调试的环境下，利用gdbserver或者别的stub得到测试结果
- 需要在测试的时候，知道远程gdbserver或者stub的机器地址和端口信息，
- 对于某些特殊的环境，需要在启动测试的时候，启动别的程序，
- 等等

从上边的列表我们能够看出，凡是和定制gdb测试过程的逻辑大多和board file有关。在我们每次运行gdb的测试用例的时候，所使用的board file绝对不止一个。往往会用到好几个board file，他们之间的关系类似面向对象中的类的继承关系，下层（孩子）的board file可以覆盖 上层（父亲）的board file中的一些函数，以实现定制功能。比如，基本的board file，就定义了测试的一般流程，比如 gdb\\\_start 启动 gdb，gdb\\\_load 加载。如果是用gdbserver进行远程调试，board file中就可以覆盖缺省的gdb\\\_start动作，加入一些启动gdbserver的 逻辑。就是利用这样的方式，dejagnu可以支持gdb的各种不同的环境下进行所有的测试，而测试用例本身不用十分在意当前的运行环境是怎样的。

这里给一个例子：

\# gdbserver running over ssh.
load\_generic\_config "gdbserver"
process\_multilib\_options ""
 
\# 设置编译测试用例的gcc
set\_board\_info compiler "/home/yao/toolchain/bin/arm-unknown-linux-gnueabi-gcc"
 
set\_board\_info rsh\_prog /usr/bin/ssh
set\_board\_info rcp\_prog /usr/bin/scp
set\_board\_info protocol standard
\# 这里进行远程调试，目标机器的hostname或者ip地址
set\_board\_info hostname your.target.host.name or ip address
set\_board\_info username yao
 
\# gdbserver's location on your target board.
set\_board\_info gdb\_server\_prog /home/yao/gdbserver
\# We will be using the standard GDB remote protocol
set\_board\_info gdb\_protocol "remote"
\# Use techniques appropriate to a stub
set\_board\_info use\_gdb\_stub 1
\# This gdbserver can only run a process once per session.
set\_board\_info gdb,do\_reload\_on\_run 1
\# There's no support for argument-passing (yet).
set\_board\_info noargs 1
\# Can't do input (or output) in the current gdbserver.
set\_board\_info gdb,noinferiorio 1
\# Can't do hardware watchpoints, in general
set\_board\_info gdb,no\_hardware\_watchpoints 1
