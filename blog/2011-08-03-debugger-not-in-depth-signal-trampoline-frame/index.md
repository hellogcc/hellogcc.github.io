---
title: "Debugger Not In Depth: signal trampoline frame"
date: "2011-08-03"
categories: 
  - "gdb"
---

在这个部分,我们要介绍signal trampoline frame对调试过程的影响,以及调试器如何处理signal trampoline frame。

**1\. signal trampoline**

当程序被信号中断,它的状态会被保存,当信号处理函数执行完毕以后,程序可以恢复到它之前被中断的点上继续执行。这就意味着从信号处理函数返回要比从一般的函数返回复杂。Linux内核安排信号处理函数在返回的时候,跳转到一小段代码,这段代码会最终执行sigreturn或者rt sigreturn, 来执行到程序之前被中断的点。这一小段代码就叫做signal trampoline。

这样说听着有些玄妙,我们看看内核是如何使用signal trampoline, 并保证信号处理函数返回到signal trampoline。

<table><tbody><tr><td><pre>1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
</pre></td><td><pre style="font-family:monospace"><span style="color:#808080;font-style:italic">/*
* handle the actual delivery of a signal to userspace
*/</span>
<span style="color:#993333">static</span> <span style="color:#993333">int</span> handle_signal<span style="color:#009900">(</span><span style="color:#993333">int</span> sig<span style="color:#339933">,</span> siginfo_t <span style="color:#339933">*</span>info<span style="color:#339933">,</span> <span style="color:#993333">struct</span> k_sigaction <span style="color:#339933">*</span>ka<span style="color:#339933">,</span>
                         sigset_t <span style="color:#339933">*</span>oldset<span style="color:#339933">,</span> <span style="color:#993333">struct</span> pt_regs <span style="color:#339933">*</span>regs<span style="color:#339933">,</span>
<span style="color:#993333">int</span> syscall<span style="color:#009900">)</span>
<span style="color:#009900">{</span>
<span style="color:#808080;font-style:italic">/* ... */</span>
ret <span style="color:#339933">=</span> setup_frame<span style="color:#009900">(</span>sig<span style="color:#339933">,</span> ka<span style="color:#339933">,</span> oldset<span style="color:#339933">,</span> regs<span style="color:#009900">)</span><span style="color:#339933">;</span>
<span style="color:#009900">}</span>
&nbsp;
<span style="color:#993333">static</span> <span style="color:#993333">int</span> setup_frame<span style="color:#009900">(</span><span style="color:#993333">int</span> signr<span style="color:#339933">,</span> <span style="color:#993333">struct</span> k_sigaction <span style="color:#339933">*</span>ka<span style="color:#339933">,</span>
sigset_t <span style="color:#339933">*</span>set<span style="color:#339933">,</span> <span style="color:#993333">struct</span> pt_regs <span style="color:#339933">*</span>regs<span style="color:#009900">)</span>
<span style="color:#009900">{</span>
  <span style="color:#808080;font-style:italic">/* ... */</span>
  retcode <span style="color:#339933">=</span> <span style="color:#009900">(</span><span style="color:#993333">unsigned</span> <span style="color:#993333">long</span> <span style="color:#339933">*</span><span style="color:#009900">)</span> frame<span style="color:#339933">-&gt;;</span>retcode<span style="color:#339933">;</span>
  put_user<span style="color:#009900">(</span>0x00003BAAUL<span style="color:#339933">,</span> retcode<span style="color:#339933">++</span><span style="color:#009900">)</span><span style="color:#339933">;</span> <span style="color:#808080;font-style:italic">/* MVK 139,B0 ; __NR_rt_sigreturn in B0 */</span>
  put_user<span style="color:#009900">(</span>0x10000000UL<span style="color:#339933">,</span> retcode<span style="color:#339933">++</span><span style="color:#009900">)</span><span style="color:#339933">;</span> <span style="color:#808080;font-style:italic">/* SWE */</span>
  put_user<span style="color:#009900">(</span>0x00006000UL<span style="color:#339933">,</span> retcode<span style="color:#339933">++</span><span style="color:#009900">)</span><span style="color:#339933">;</span> <span style="color:#808080;font-style:italic">/* NOP 4 */</span>
  put_user<span style="color:#009900">(</span>0x00006000UL<span style="color:#339933">,</span> retcode<span style="color:#339933">++</span><span style="color:#009900">)</span><span style="color:#339933">;</span> <span style="color:#808080;font-style:italic">/* NOP 4 */</span>
  put_user<span style="color:#009900">(</span>0x00006000UL<span style="color:#339933">,</span> retcode<span style="color:#339933">++</span><span style="color:#009900">)</span><span style="color:#339933">;</span> <span style="color:#808080;font-style:italic">/* NOP 4 */</span>
  put_user<span style="color:#009900">(</span>0x00006000UL<span style="color:#339933">,</span> retcode<span style="color:#339933">++</span><span style="color:#009900">)</span><span style="color:#339933">;</span> <span style="color:#808080;font-style:italic">/* NOP 4 */</span>
  put_user<span style="color:#009900">(</span>0x00006000UL<span style="color:#339933">,</span> retcode<span style="color:#339933">++</span><span style="color:#009900">)</span><span style="color:#339933">;</span> <span style="color:#808080;font-style:italic">/* NOP 4 */</span>
&nbsp;
  <span style="color:#808080;font-style:italic">/* Change user context to branch to signal handler */</span>
  regs<span style="color:#339933">-&gt;</span>sp <span style="color:#339933">=</span> <span style="color:#009900">(</span><span style="color:#993333">unsigned</span> <span style="color:#993333">long</span><span style="color:#009900">)</span> frame <span style="color:#339933">-</span> <span style="color:#0000dd">8</span><span style="color:#339933">;</span>
  regs<span style="color:#339933">-&gt;</span>b3 <span style="color:#339933">=</span> <span style="color:#009900">(</span><span style="color:#993333">unsigned</span> <span style="color:#993333">long</span><span style="color:#009900">)</span> retcode<span style="color:#339933">;</span>
  regs<span style="color:#339933">-&gt;</span>pc <span style="color:#339933">=</span> <span style="color:#009900">(</span><span style="color:#993333">unsigned</span> <span style="color:#993333">long</span><span style="color:#009900">)</span> ka<span style="color:#339933">-&gt;</span>sa.<span style="color:#202020">sa_handler</span><span style="color:#339933">;</span>
&nbsp;
  <span style="color:#808080;font-style:italic">/* Give the signal number to the handler */</span>
  regs<span style="color:#339933">-&gt;</span>a4 <span style="color:#339933">=</span> signr<span style="color:#339933">;</span>
  regs<span style="color:#339933">-&gt;</span>b4 <span style="color:#339933">=</span> <span style="color:#009900">(</span><span style="color:#993333">unsigned</span> <span style="color:#993333">long</span><span style="color:#009900">)</span> <span style="color:#339933">&amp;</span>amp<span style="color:#339933">;</span>frame<span style="color:#339933">-&gt;</span>sc<span style="color:#339933">;</span>
&nbsp;
  <span style="color:#808080;font-style:italic">/* ... */</span>
<span style="color:#009900">}</span></pre></td></tr></tbody></table>

我们看到函数handle signal调用了setup frame,其中setup frame设置了使得信号处理函数返回到signal trampoline (函数返回值的地址在寄存器B3)。这样当信号处理函数执行完毕,就会自动的转到signaltrampoline,最后调用sigreturn返回。这里可以看出,signal trampoline完全处于正常程序和信号处理函数之间,如果调试器不能正确识别signal trampoline,很多调式功能在信号处理函数上,就会有问题。

**2\. signal trampoline给调试带来的问题**

在知道了signal trampoline是什么以后,我们来描述一下signal trampoline给调式带来的问题。例如我们现在有个正常函数func,在执行到一半的时候,受到信号, 进而执行信号处理函数handler, 假如我们在handler上设置了断点,有一些调试操作是不能正确执行的,

1\. 查看函数调用堆栈。实际上，当前的函数调用栈包括：正常函数func，signaltrampoline的一些信息，信号处理函数handler。其中func和handler都是一般的函数 ，所以它们的frame都可以容易处理，但是signal trampoline是很特殊的一段代码，它的frame不能按照通常的frame来处理。

2\. 命令next。命令next的意思是执行程序到下一行，如果程序当前已经在函数最后一行，那么这个命令应该使得程序执行一段停止在调用这个函数的下一句。这些是我们在调试器手册看到的解释，但是调试器内部怎么理解这样的条件呢？调试器会根据不同frame之间的关系(inner than 或者 outter than)，来判断这些条件（调试器内部的frame管理，是一个很复杂的模块，对于不熟悉frame和unwinding的读者，可以暂时忽略frame和unwinding的内部原理）。如果调试器不能识别signal trampoline，这些命令在信号处理函数的尾部，就无法使用。

**3 支持signal trampoline**

在上边分析完signal trampoline给调试带来的问题后，我们把支持signal trampoline总结为两个部分

1. 识别signal trampoline。调试器首先能够在代码里边识别signal trampoline。因为signal trampoline都是很特别而且很短的代码，所以调试器可以匹配代码指令，就能找到signal trampoline。
2.  unwind signal trampoline frame。我们上边说过，signal trampoline的frame联系着正常函数和信号处理函数，如果调试器希望在堆栈上正确的找到他们各自的内容，就需要正确分析signal trampoline的frame（对于不清楚frame unwinding的读者，可以暂时认为frame unwinding就是递归地找到当前函数的caller的寄存器保存的位置和值）。所以，对于signal trampoline frame，我们也需要找到保存它的caller（由于从signal trampoline后就进入sigreturn，进而回到原来正常函数，我们可以认为signal trampoline的caller就是原来的正常函数）的寄存器的位置。

识别signal trampoline就是一个对指令进行模式匹配的过程，如果指令匹配上了一个模式，这个几条指令就是signal trampoline。如所示，我们可以把指令模式定义如下，

我这里不打算具体去讲这个结构的定义和含义（结构tramp\_frame也是GDB内部的数据结构），只展示如何定义自己的指令模式，以识别signal trampoline。在定义完这个指令模式后，第一个问题就解决了，下来我们看第二个问题，如何找到caller的寄存器的位置和值  
。当函数执行收到信号的时候，内核会把进程当前状态保存在堆栈上某个位置，进而在恢复的，从堆栈上得到。我们这里的工作就是，查看内核源代码，得到寄存器的保存位置，然后我们就可以得到这些寄存器的值。在这里，“获得寄存器的值”还是有些模糊的，在这个环境中，我们可以把获得寄存器分为两个部分：sp，fp和其他寄存器。在函数调用的过程中，寄存器都是保存在堆栈上，所以基本都是基于sp或者fp的寻址。

我们结合代码来看看如何得到signal trampoline frame上保存的寄存器。前提是我们已经知道sp，下来就是寻找保存寄存器的位置（在堆栈上）对sp的偏移。

<table><tbody><tr><td><pre>1
2
3
4
5
6
7
</pre></td><td><pre style="font-family:monospace">asmlinkage <span style="color:#993333">int</span> do_rt_sigreturn<span style="color:#009900">(</span><span style="color:#993333">struct</span> pt_regs <span style="color:#339933">*</span>regs<span style="color:#009900">)</span>
<span style="color:#009900">{</span>
  <span style="color:#993333">struct</span> rt_sigframe <span style="color:#339933">*</span>frame<span style="color:#339933">;&lt;/</span>code<span style="color:#339933">&gt;</span>
&nbsp;
  frame <span style="color:#339933">=</span> <span style="color:#009900">(</span><span style="color:#993333">struct</span> rt_sigframe <span style="color:#339933">*</span><span style="color:#009900">)</span> <span style="color:#009900">(</span><span style="color:#009900">(</span><span style="color:#993333">unsigned</span> <span style="color:#993333">long</span><span style="color:#009900">)</span> regs<span style="color:#339933">-&gt;</span>sp <span style="color:#339933">+</span> <span style="color:#0000dd">8</span><span style="color:#009900">)</span><span style="color:#339933">;</span>
<span style="color:#808080;font-style:italic">/* ... */</span>
<span style="color:#009900">}</span></pre></td></tr></tbody></table>

从上边的代码中我们能看到，rt\_sigframe的起始地址距离sp为8，我们接着看rt\_sigframe的结构。

<table><tbody><tr><td><pre>1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
</pre></td><td><pre style="font-family:monospace"><span style="color:#993333">struct</span> rt_sigframe
<span style="color:#009900">{</span>
  <span style="color:#993333">struct</span> siginfo <span style="color:#339933">*</span>pinfo<span style="color:#339933">;</span>
  <span style="color:#993333">void</span> <span style="color:#339933">*</span>puc<span style="color:#339933">;</span>
  <span style="color:#993333">struct</span> siginfo info<span style="color:#339933">;</span>
  <span style="color:#993333">struct</span> ucontext uc<span style="color:#339933">;</span>
  <span style="color:#993333">unsigned</span> <span style="color:#993333">long</span> retcode<span style="color:#009900">[</span>RETCODE_SIZE <span style="color:#339933">&gt;&gt;</span> <span style="color:#0000dd">2</span><span style="color:#009900">]</span><span style="color:#339933">;</span>
<span style="color:#009900">}</span><span style="color:#339933">;</span>
&nbsp;
<span style="color:#993333">struct</span> ucontext <span style="color:#009900">{</span>
  <span style="color:#993333">unsigned</span> <span style="color:#993333">long</span> uc_flags<span style="color:#339933">;</span>
  <span style="color:#993333">struct</span> ucontext <span style="color:#339933">*</span>uc_link<span style="color:#339933">;</span>
  stack_t uc_stack<span style="color:#339933">;</span>
  <span style="color:#993333">struct</span> sigcontext uc_mcontext<span style="color:#339933">;</span>
  sigset_t uc_sigmask<span style="color:#339933">;</span> <span style="color:#808080;font-style:italic">/* mask last for extensibility */</span>
<span style="color:#009900">}</span><span style="color:#339933">;</span></pre></td></tr></tbody></table>

最后，我们发现寄存器都保存在结构struct sigcontext中。这样我们就可以计算出保存每个寄存器距离sp的偏移，然后从这些地址读出寄存器的值，填写每个frame，

trad\_frame\_set\_reg\_addr (this\_cache, reg\_num,
base + foo\_register\_sigcontext\_offset (reg\_num));

这样，一个完整的signal trampoline frame unwinding就做好了。
