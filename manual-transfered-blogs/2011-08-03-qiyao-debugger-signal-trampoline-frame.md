原始博客链接：

转换时间：
2021-01-31

注意：由于2011年HelloGCC博客遭遇了一次VPS掉电丢失，所有2011年之前（含）的博客都是从 Google Reader 等RSS中恢复的。损失了大部分的内容，非常的遗憾。如果你的电脑中保存过HelloGCC2012年之前的报告，欢迎发送给我们，非常感谢。请转发给 hellogcc.workgroup@gmail.com

# Debugger Not In Depth: signal trampoline frame
Posted on 2011/08/03 by yao

在这个部分,我们要介绍signal trampoline frame对调试过程的影响,以及调试器如何处理signal trampoline frame。

1. signal trampoline

当程序被信号中断,它的状态会被保存,当信号处理函数执行完毕以后,程序可以恢复到它之前被中断的点上继续执行。这就意味着从信号处理函数返回要比从一般的函数返回复杂。Linux内核安排信号处理函数在返回的时候,跳转到一小段代码,这段代码会最终执行sigreturn或者rt sigreturn, 来执行到程序之前被中断的点。这一小段代码就叫做signal trampoline。

这样说听着有些玄妙,我们看看内核是如何使用signal trampoline, 并保证信号处理函数返回到signal trampoline。

```
/*
* handle the actual delivery of a signal to userspace
*/
static int handle_signal(int sig, siginfo_t *info, struct k_sigaction *ka,
                         sigset_t *oldset, struct pt_regs *regs,
int syscall)
{
/* ... */
ret = setup_frame(sig, ka, oldset, regs);
}

static int setup_frame(int signr, struct k_sigaction *ka,
sigset_t *set, struct pt_regs *regs)
{
  /* ... */
  retcode = (unsigned long *) frame->;retcode;
  put_user(0x00003BAAUL, retcode++); /* MVK 139,B0 ; __NR_rt_sigreturn in B0 */
  put_user(0x10000000UL, retcode++); /* SWE */
  put_user(0x00006000UL, retcode++); /* NOP 4 */
  put_user(0x00006000UL, retcode++); /* NOP 4 */
  put_user(0x00006000UL, retcode++); /* NOP 4 */
  put_user(0x00006000UL, retcode++); /* NOP 4 */
  put_user(0x00006000UL, retcode++); /* NOP 4 */

  /* Change user context to branch to signal handler */
  regs->sp = (unsigned long) frame - 8;
  regs->b3 = (unsigned long) retcode;
  regs->pc = (unsigned long) ka->sa.sa_handler;

  /* Give the signal number to the handler */
  regs->a4 = signr;
  regs->b4 = (unsigned long) &amp;frame->sc;

  /* ... */
}
```
我们看到函数 handle signal 调用了setup frame,其中setup frame设置了使得信号处理函数返回到signal trampoline (函数返回值的地址在寄存器B3)。这样当信号处理函数执行完毕,就会自动的转到signaltrampoline,最后调用sigreturn返回。这里可以看出,signal trampoline完全处于正常程序和信号处理函数之间,如果调试器不能正确识别signal trampoline,很多调式功能在信号处理函数上,就会有问题。

2. signal trampoline 给调试带来的问题

在知道了signal trampoline是什么以后,我们来描述一下signal trampoline给调式带来的问题。例如我们现在有个正常函数func,在执行到一半的时候,受到信号, 进而执行信号处理函数handler, 假如我们在handler上设置了断点,有一些调试操作是不能正确执行的,

    1. 查看函数调用堆栈。实际上，当前的函数调用栈包括：正常函数func，signaltrampoline的一些信息，信号处理函数handler。其中func和handler都是一般的函数 ，所以它们的frame都可以容易处理，但是signal trampoline是很特殊的一段代码，它的frame不能按照通常的frame来处理。

    2. 命令next。命令next的意思是执行程序到下一行，如果程序当前已经在函数最后一行，那么这个命令应该使得程序执行一段停止在调用这个函数的下一句。这些是我们在调试器手册看到的解释，但是调试器内部怎么理解这样的条件呢？调试器会根据不同frame之间的关系(inner than 或者 outter than)，来判断这些条件（调试器内部的frame管理，是一个很复杂的模块，对于不熟悉frame和unwinding的读者，可以暂时忽略frame和unwinding的内部原理）。如果调试器不能识别signal trampoline，这些命令在信号处理函数的尾部，就无法使用。

3 支持signal trampoline

在上边分析完signal trampoline给调试带来的问题后，我们把支持signal trampoline总结为两个部分

    识别signal trampoline。调试器首先能够在代码里边识别signal trampoline。因为signal trampoline都是很特别而且很短的代码，所以调试器可以匹配代码指令，就能找到signal trampoline。
     unwind signal trampoline frame。我们上边说过，signal trampoline的frame联系着正常函数和信号处理函数，如果调试器希望在堆栈上正确的找到他们各自的内容，就需要正确分析signal trampoline的frame（对于不清楚frame unwinding的读者，可以暂时认为frame unwinding就是递归地找到当前函数的caller的寄存器保存的位置和值）。所以，对于signal trampoline frame，我们也需要找到保存它的caller（由于从signal trampoline后就进入sigreturn，进而回到原来正常函数，我们可以认为signal trampoline的caller就是原来的正常函数）的寄存器的位置。

识别signal trampoline就是一个对指令进行模式匹配的过程，如果指令匹配上了一个模式，这个几条指令就是signal trampoline。如所示，我们可以把指令模式定义如下，

我这里不打算具体去讲这个结构的定义和含义（结构tramp_frame也是GDB内部的数据结构），只展示如何定义自己的指令模式，以识别signal trampoline。在定义完这个指令模式后，第一个问题就解决了，下来我们看第二个问题，如何找到caller的寄存器的位置和值
。当函数执行收到信号的时候，内核会把进程当前状态保存在堆栈上某个位置，进而在恢复的，从堆栈上得到。我们这里的工作就是，查看内核源代码，得到寄存器的保存位置，然后我们就可以得到这些寄存器的值。在这里，“获得寄存器的值”还是有些模糊的，在这个环境中，我们可以把获得寄存器分为两个部分：sp，fp和其他寄存器。在函数调用的过程中，寄存器都是保存在堆栈上，所以基本都是基于sp或者fp的寻址。

我们结合代码来看看如何得到signal trampoline frame上保存的寄存器。前提是我们已经知道sp，下来就是寻找保存寄存器的位置（在堆栈上）对sp的偏移。

```
asmlinkage int do_rt_sigreturn(struct pt_regs *regs)
{
  struct rt_sigframe *frame;</code>

  frame = (struct rt_sigframe *) ((unsigned long) regs->sp + 8);
/* ... */
}
```
从上边的代码中我们能看到，rt_sigframe的起始地址距离sp为8，我们接着看rt_sigframe的结构。
```
struct rt_sigframe
{
  struct siginfo *pinfo;
  void *puc;
  struct siginfo info;
  struct ucontext uc;
  unsigned long retcode[RETCODE_SIZE >> 2];
};

struct ucontext {
  unsigned long uc_flags;
  struct ucontext *uc_link;
  stack_t uc_stack;
  struct sigcontext uc_mcontext;
  sigset_t uc_sigmask; /* mask last for extensibility */
};
```
最后，我们发现寄存器都保存在结构struct sigcontext中。这样我们就可以计算出保存每个寄存器距离sp的偏移，然后从这些地址读出寄存器的值，填写每个frame，
```
trad_frame_set_reg_addr (this_cache, reg_num,
base + foo_register_sigcontext_offset (reg_num));
```
这样，一个完整的signal trampoline frame unwinding就做好了。
