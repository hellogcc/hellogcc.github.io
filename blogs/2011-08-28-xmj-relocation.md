原始博客链接：

转换时间：
2021-01-31

注意：由于2011年HelloGCC博客遭遇了一次VPS掉电丢失，所有2011年之前（含）的博客都是从 Google Reader 等RSS中恢复的。损失了大部分的内容，非常的遗憾。如果你的电脑中保存过HelloGCC2012年之前的报告，欢迎发送给我们，非常感谢。请转发给 hellogcc.workgroup@gmail.com

# 一个关于重定位的例子
Posted on 2011/08/28 by xmj

xmj@hellogcc

参考文档：SYSTEM V APPLICATION BINARY INTERFACE

重定位（relocation），可以理解为有些内容在编译和汇编的时候不能确定下来，需要在连接成可执行程序时才能够计算求得。最常见的就是跳转指令中的目标地址。

ELF文件中会有一些重定位段，比如对应于.text段的.rel.text段。重定位段中的每一项，即重定位项，描述了其对应的需要被重定位的地方，以及如何进行求值。

1、重定位项有两种结构类型，

```
typedef struct {
  Elf32_Addr r_offset;
  Elf32_Word r_info;
} Elf32_Rel;

typedef struct {
  Elf32_Addr r_offset;
  Elf32_Word r_info;
  Elf32_Sword r_addend;
} Elf32_Rela;
```

`r_offset` ，需要进行重定位的地方，到其所在段开始处的字节偏移量；

`r_info` ，有两部分组成，一部分是所对应的符号表中的索引，一部分是重定位类型；它们的组合方式如下：


```
#define ELF32_R_SYM(i) ((i)>>8)
#define ELF32_R_TYPE(i) ((unsigned char)(i))
#define ELF32_R_INFO(s,t) (((s)<<8)+(unsigned char)(t))
```
r_addend，计算求值时的常量加数。

MIPS的重定位项是使用了第一种方式，这种方式是将r_addend的值存放在了将要被修改的位置，也就是目标文件中需要被重定位的地方的初始内容。

2、执行如下命令，

```
$ mipsel-linux-readelf -r sysconf.o

Relocation section ‘.rel.rodata’ at offset 0xb74 contains 132 entries:
Offset Info Type Sym.Value Sym. Name
00000000 0000020c R_MIPS_GPREL32 00000000 .text
00000004 0000020c R_MIPS_GPREL32 00000000 .text
00000008 0000020c R_MIPS_GPREL32 00000000 .text
0000000c 0000020c R_MIPS_GPREL32 00000000 .text
00000010 0000020c R_MIPS_GPREL32 00000000 .text
…
```
这里，打印出了每个重定位项的offset和info。从info中的内容，解析出了type和sym索引，进一步解析出sym.name和sym.value。可以看到，除了.text段中有需要进行重定位的地方以外，.rodata中也有一些，注意，这些offset是对应于.rodata段。

3、执行如下命令，

```
$ mipsel-linux-objdump -Dr sysconf.o

Disassembly of section .rodata:

00000000 <.rodata>:
0: ffffc064 0xffffc064
0: R_MIPS_GPREL32 .text
4: ffffc06c 0xffffc06c
4: R_MIPS_GPREL32 .text
8: ffffc074 0xffffc074
8: R_MIPS_GPREL32 .text
c: ffffc07c 0xffffc07c
c: R_MIPS_GPREL32 .text
10: ffffc084 0xffffc084
10: R_MIPS_GPREL32 .text
…
```
这里将.rodata中需要重定位的地方和对应的重定位项结合在一起打印出来。可以看出，对于.rodata中的第一个字，其保存的addend值为0xffffc064。

4、MIPS ABI文档如下描述R_MIPS_GPREL_32：
```
Name Value Field Symbol Calculation
R_MIPS_GPREL_32 12 T-word32 local A + S + GP0 - GP

A
Represents the addend used to compute the value of the relocatable
field.

S
Represents the value of the symbol whose index resides in the relocation entry, unless the the symbol is STB_LOCAL and is of type
STT_SECTION in which case S represents the original sh_addr minus
the final sh_addr.

GP
Represents the final gp value to be used for the relocatable, executable, or shared object file being produced.

GP0
Represents the gp value used to create the relocatable object.

这里的“A + S + GP0 – GP”便是连接器进行重定位时的求值公式。
```
5、执行如下命令
```
$ mipsel-linux-objdump -Dr sysconf.o

Disassembly of section .reginfo:

00000000 <.reginfo>:
0: b200001e 0xb200001e
…
14: 00004000 sll t0,zero,0x0
```
MIPS通过reginfo保存一些寄存器信息，其结构体为：

```
typedef struct {
  Elf32_Word ri_gprmask;
  Elf32_Word ri_cprmask[4];
  Elf32_SWord ri_gp_value;
} ELF_RegInfo;
```
其中最后一个域是记录了GP0的值。可以看到，这里GP0的值为0×4000。

6、当链接成可执行程序时，依照同样的方法，我们可以找到

```
00400094 <.reginfo>:
400094: b20000f4 0xb20000f4
...
4000a8: 10008a70 b 3e2a6c <__start-0x1d6a4>
```
所以，这里GP的值为0x10008a70，

```
00425c70 <__sysconf>:
425c70: 3c1c0fbe lui gp,0xfbe
```
所以，S的值为0x425c70，

7、根据公式“A + S + GP0 – GP”，可以计算出.rodata中需要重定位的地方的最终内容。比如，对于.rodata中的第一个字，

```
0xffffc064 + 0x425c70 + 0x4000 - 0x10008a70 = 0xf041d264
```
8、附注

如果查看一下binutils/bfd/elf32-mips.c，就可以找到R_MIPS_GPREL32相应的HOWTO数据结构，以及处理函数。
