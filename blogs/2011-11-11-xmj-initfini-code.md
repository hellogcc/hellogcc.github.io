原始博客链接：

转换时间：
2021-01-31

注意：由于2011年HelloGCC博客遭遇了一次VPS掉电丢失，所有2011年之前（含）的博客都是从 Google Reader 等RSS中恢复的。损失了大部分的内容，非常的遗憾。如果你的电脑中保存过HelloGCC2012年之前的报告，欢迎发送给我们，非常感谢。请转发给 hellogcc.workgroup@gmail.com

# 有趣的代码——initfini.c
Posted on 2011/11/11 by xmj

xmj@hellogcc

1、最近编译glibc库，在链接时遇到一个问题，现象如下：
```
crtn.S:20: multiple definition of `dummy'
crtn.S:37: multiple definition of `_init'
crtn.S:88: multiple definition of `_fini'
crti.S:58: undefined reference to `i_am_not_a_leaf'
crtn.S:110: undefined reference to `i_am_not_a_leaf'
crtn.S:111: undefined reference to `i_am_not_a_leaf'
```
搜了下，按照网上的提示把问题解决了。顺便看下initfini.c的代码，感觉挺有意思。

2、这个C文件（initfini.c），首先会被编译成s文件（initfini.s），然后再使用sed程序将其分割成两个S文件（crti.S和crtn.S），最后分别编译成crti.o和crtn.o。文件开头注释里介绍了crti.o和crtn.o的作用：



```
# This directory contains the C startup code (that which calls main).  This
# consists of the startfile, built from start.c and installed as crt0.o
# (traditionally) or crt1.o (for ELF).  In ELF we also install crti.o and
# crtn.o, special "initializer" and "finalizer" files used in the link
# to make the .init and .fini sections work right; both these files are
# built (in an arcane manner) from initfini.c.
```

Makefile里，有对应的sed程序：
```
# We only have one kind of startup code files.  Static binaries and
# shared libraries are build using the PIC version.
$(objpfx)crti.S: $(objpfx)initfini.s
       sed -n -e '1,/@HEADER_ENDS/p' \
              -e '/@_.*_PROLOG_BEGINS/,/@_.*_PROLOG_ENDS/p' \
              -e '/@TRAILER_BEGINS/,$$p' $< > $@

$(objpfx)crtn.S: $(objpfx)initfini.s
       sed -n -e '1,/@HEADER_ENDS/p' \
              -e '/@_.*_EPILOG_BEGINS/,/@_.*_EPILOG_ENDS/p' \
              -e '/@TRAILER_BEGINS/,$$p' $< > $@
```

对照着C代码和生成的汇编代码，就可以看出，它的实现方法是，用sed通过C代码中特定的注释来提取、分割汇编代码的。比如“@HEADER_ENDS”，在C文件中的代码如下：



```
/* The initial common code ends here. */
asm ("\n/*@HEADER_ENDS*/");
```
生成的汇编代码如下：
```
/*@HEADER_ENDS*/
```


3、显然，这些嵌入的注释，在生成的汇编代码里需要保持原有的顺序，这样才能保证sed可以正确的提取汇编代码。

问题就出在gcc恰恰把生成汇编代码中的注释顺序打乱了，导致sed没能正确提取汇编代码，生成了错误的crti.S和crtn.S。网上的解决方法是，关掉一个优化选项。将Makefile中的
```
CFLAGS-initfini.s = -g0 -fPIC -fno-inline-functions
```
改为
```
CFLAGS-initfini.s = -g0 -fPIC -fno-inline-functions -fno-unit-at-a-time
```
即可。
