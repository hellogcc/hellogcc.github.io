---
title: "PLT实例讲解"
date: "2012-05-14"
categories: 
  - "其它技术"
---

by xmj, yao

一、x86 ABI手册原文及翻译

原文摘自SYSTEM V APPLICATION BINARY INTERFACE。

Figure 5-7: Position-Independent Procedure Linkage Table

.PLT0: pushl 4(%ebx)
       jmp \*8(%ebx)
       nop; nop
       nop; nop
.PLT1: jmp \*name1@GOT(%ebx)
       pushl $offset
       jmp .PLT0@PC
.PLT2: jmp \*name2@GOT(%ebx)
       pushl $offset
       jmp .PLT0@PC
...

Following the steps below, the dynamic linker and the program ‘‘cooperate’’ to resolve symbolic references through the procedure linkage table and the global offset table.

动态链接器和程序，按照下面的步骤，协作完成对通过过程链接表和全局偏移表进行符号引用的解析。

1 . When first creating the memory image of the program, the dynamic linker sets the second and the third entries in the global offset table to special values. Steps below explain more about these values.

动态链接器在开始创建程序的内存映像时，会将全局偏移表中的第二，三项设置为特定的值。这些值在下面的步骤中详细解释。

2 . If the procedure linkage table is position-independent, the address of the global offset table must reside in %ebx. Each shared object file in the process image has its own procedure linkage table, and control transfers to a procedure linkage table entry only from within the same object file. Consequently, the calling function is responsible for setting the global offset table base register before calling the procedure linkage table entry.

如果过程链接表是位置无关的，则全局偏移表的地址必须存在%ebx中。进程映像中的每个共享目标文件都有自己的过程链接表，并且只能从同一个目标文件中才能将控制转换到过程链接表的表项。因此，调用函数需要在调用过程链接表项之前，设置全局偏移表的基础寄存器。

3 . For illustration, assume the program calls name1, which transfers control to the label .PLT1.

例如，假设程序调用了name1，其将控制转换到标号.PLT1.

4 . The first instruction jumps to the address in the global offset table entry for name1. Initially, the global offset table holds the address of the following  
pushl instruction, not the real address of name1.

第一条指令跳转到全局偏移表项中name1的地址。初始的时候，全局偏移表中存放的是pushl指令之后的地址，而不是name1的实际地址。

5 . Consequently, the program pushes a relocation offset (offset) on the stack. The relocation offset is a 32-bit, non-negative byte offset into the relocation table. The designated relocation entry will have type R\_386\_JMP\_SLOT, and its offset will specify the global offset table entry used in the previous jmp instruction. The relocation entry also contains a symbol table index, thus telling the dynamic linker what symbol is being referenced, name1 in this case.

因此，程序将一个重定位偏移量（offset）压入栈中。重定位偏移量为一个32位，非负的，重定位表的字节偏移。其所指定的重定位项将具有R\_386\_JMP\_SLOT类型，并且它的偏移量指定了在之前jmp指令中会用到的全局偏移表项。重定位项还包含了一个符号表索引，因此告诉了动态链接器哪个符号在被引用。在该例子中，为name1.

6 . After pushing the relocation offset, the program then jumps to .PLT0, the first entry in the procedure linkage table. The pushl instruction places the value of the second global offset table entry (got\_plus\_4 or 4(%ebx)) on the stack, thus giving the dynamic linker one word of identifying information. The program then jumps to the address in the third global offset table entry (got\_plus\_8 or 8(%ebx)), which transfers control to the dynamic linker.

在压入重定位偏移量之后，程序然后跳转到.PLT0，过程链接表的第一项。pushl指令将全局偏移表的第二个表项（got\_plus\_4 or 4(%ebx)）压入栈中，因此给了动态链接器一个字的标识信息。程序然后跳转到全局偏移表的第三个表项中（got\_plus\_8 or 8(%ebx)）的地址，其将控制转换给动态链接器。

7 . When the dynamic linker receives control, it unwinds the stack, looks at the designated relocation entry, finds the symbol’s value, stores the ‘‘real’’ address for name1 in its global offset table entry, and transfers control to the desired destination.

当动态链接器获得控制之后，其展开栈，查看指定的重定位项，发现符号的值，将name1的“实际”地址存放在它的全局偏移表项中，然后将控制转换到所希望的目的地。

8 . Subsequent executions of the procedure linkage table entry will transfer directly to name1, without calling the dynamic linker a second time. That is, the jmp instruction at .PLT1 will transfer to name1, instead of ‘‘falling through’’ to the pushl instruction.

以后对过程链接表项的执行，将会直接转换到name1，而不需要再次调用动态链接器。也就是说，在.PLT1中的jmp指令会直接跳转到name1，而不会顺序执行到pushl指令。

二、实例分析

1、为了帮助理解这些枯燥的文档，我们结合一个实际的例子进行分析。

例子很简单，

#include 

int
main (void)
{
 printf ("hellogcc\\n");

 return 0;
}

2、后边我们会看到一些汇编程序和一些地址，为了搞清楚这些地址的含义，我们先列出一些段的地址范围，

(gdb) maintenance info sections
Exec file:
   \`/home/yao/SourceCode/plt.exe', file type elf32-i386.
   0x80481d4->0x8048224 at 0x000001d4: .dynsym ALLOC LOAD READONLY DATA HAS\_CONTENTS
   0x8048224->0x804826e at 0x00000224: .dynstr ALLOC LOAD READONLY DATA HAS\_CONTENTS
   0x8048298->0x80482a0 at 0x00000298: .rel.dyn ALLOC LOAD READONLY DATA HAS\_CONTENTS
   0x80482a0->0x80482b8 at 0x000002a0: .rel.plt ALLOC LOAD READONLY DATA HAS\_CONTENTS
   0x80482b8->0x80482e8 at 0x000002b8: .init ALLOC LOAD READONLY CODE HAS\_CONTENTS
   0x80482e8->0x8048328 at 0x000002e8: .plt ALLOC LOAD READONLY CODE HAS\_CONTENTS
   0x8048330->0x804849c at 0x00000330: .text ALLOC LOAD READONLY CODE HAS\_CONTENTS
   0x804849c->0x80484b8 at 0x0000049c: .fini ALLOC LOAD READONLY CODE HAS\_CONTENTS
   0x80484b8->0x80484c9 at 0x000004b8: .rodata ALLOC LOAD READONLY DATA HAS\_CONTENTS
   0x80484cc->0x80484d0 at 0x000004cc: .eh\_frame ALLOC LOAD READONLY DATA HAS\_CONTENTS
   0x8049f0c->0x8049f14 at 0x00000f0c: .ctors ALLOC LOAD DATA HAS\_CONTENTS
   0x8049f14->0x8049f1c at 0x00000f14: .dtors ALLOC LOAD DATA HAS\_CONTENTS
   0x8049f1c->0x8049f20 at 0x00000f1c: .jcr ALLOC LOAD DATA HAS\_CONTENTS
   0x8049f20->0x8049ff0 at 0x00000f20: .dynamic ALLOC LOAD DATA HAS\_CONTENTS
   0x8049ff0->0x8049ff4 at 0x00000ff0: .got ALLOC LOAD DATA HAS\_CONTENTS
   0x8049ff4->0x804a00c at 0x00000ff4: .got.plt ALLOC LOAD DATA HAS\_CONTENTS
   0x804a00c->0x804a014 at 0x0000100c: .data ALLOC LOAD DATA HAS\_CONTENTS
   0x804a014->0x804a01c at 0x00001014: .bss ALLOC

3、我们看看实际程序中，我们PLT section里边的内容是什么？

(gdb) disassemble 0x80482e8,0x8048328
Dump of assembler code from 0x80482e8 to 0x8048328:
  0x080482e8:  pushl  0x8049ff8
  0x080482ee:  jmp    \*0x8049ffc
  0x080482f4:  add    %al,(%eax)
  0x080482f6:  add    %al,(%eax)
  0x080482f8:  jmp    \*0x804a000
  0x080482fe:  push   $0x0
  0x08048303:  jmp    0x80482e8
  0x08048308:  jmp    \*0x804a004
  0x0804830e:  push   $0x8
  0x08048313:  jmp    0x80482e8
  0x08048318:  jmp    \*0x804a008
  0x0804831e:  push   $0x10
  0x08048323:  jmp    0x80482e8

我们看到了，puts的plt entry，是 plt 3，前边的0 1 和 2都已经被占用了。这些都是系统  
保留的entry。不同的体系结构，这里可能占用不同的书目的entry。plt 0会在本文中介绍  
到，但是 plt 1 和 2 的作用，没有在本文介绍。

4、第一条指令跳转到全局偏移表项中name1的地址。初始的时候，全局偏移表中存放的是pushl指令之后的地址，而不是name1的实际地址。

  0x08048318:     jmp    \*0x804a008 // -> jmp \*(\_GLOBAL\_OFFSET\_TABLE\_+20)
  0x0804831e:     push   $0x10      // push relocation offset.

我们可以看到 0x804a008 落在的 .got.plt 的范围，

   0x8049ff4->0x804a00c at 0x00000ff4: .got.plt ALLOC LOAD DATA HAS\_CONTENTS

(gdb) x/4x 0x804a008
0x804a008 :   0x0804831e      0x00000000      0x00000000      0x00000000

5 . 因此，程序将一个重定位偏移量（offset）压入栈中 (see the insn on 0x0804831e: push 0×10)。重定位偏移量为一个32位，非负的，重定位表的字节偏移。其所指定的重定位项将具有R\_386\_JMP\_SLOT类型，并且它的偏移量指定了在之前jmp指令中会用到的全局偏移表项。

Relocation section '.rel.plt' at offset 0x2a0 contains 3 entries:
 Offset     Info    Type            Sym.Value  Sym. Name
0804a000  00000107 R\_386\_JUMP\_SLOT   00000000   \_\_gmon\_start\_\_
0804a004  00000207 R\_386\_JUMP\_SLOT   00000000   \_\_libc\_start\_main
0804a008  00000307 R\_386\_JUMP\_SLOT   00000000   puts

我们可以看到，这里有一个reloc R\_386\_JUMP\_SLOT，对应的地址是0x804a008，其实就是 puts对应的 .got.plt 的entry。

重定位项还包含了一个符号表索引，因此告诉了动态链接器哪个符号在被引用。在该例子中，为name1.

这个offset（0×10）是指的.rel.plt段的偏移，也就是第三项

0804a008  00000307 R\_386\_JUMP\_SLOT   00000000   puts

这里可以看出，.rel.plt的每一项是8个字节，我手中的这个ABI手册比较旧，没有对这个段和每一项的大小做介绍。

重定位项R\_386\_JUMP\_SLOT包含了offset，info，type，symbol这些信息。其中offset（0x0804a008）指定了在之前jmp指令中会用到的全局偏移表项，symbol信息告诉动态链接器哪个符号在被引用。动态链接器要做的事情就是将这个符号的实际值（即name1的值）填写到偏移量为0x0804a008的全局偏移表项中，即更新name1的全局偏移表项。

6 . 在压入重定位偏移量之后，程序然后跳转到.PLT0，过程链接表的第一项。

  0x08048323
:    jmp    0x80482e8  // jump to start of .plt section.

.PLT0:
  0x080482e8:  pushl  0x8049ff8
  0x080482ee:  jmp    \*0x8049ffc

pushl指令将.got.plt的第二个表项（got\_plus\_4 or 4(%ebx)）压入栈中，因此给了动态链接器一个字的标识信息。

  0x8049ff4->0x804a00c at 0x00000ff4: .got.plt ALLOC LOAD DATA HAS\_CONTENTS

程序然后跳转到.got.plt的第三个表项中（got\_plus\_8 or 8(%ebx)）的地址，其将控制转换给动态链接器。

(gdb) x/x 0x8049ffc
0x8049ffc :    0x00123270
(gdb) disassemble 0x00123270,0x00123280
Dump of assembler code from 0x123270 to 0x123280:
  0x00123270 :  push   %eax
  0x00123271 :  push   %ecx
  0x00123272 :  push   %edx
  0x00123273 :  mov    0x10(%esp),%edx
  0x00123277 :  mov    0xc(%esp),%eax
  0x0012327b : call   0x11d5a0

可以看到，\`jmp \*0x8049ffc’ 跳转到了 \_dl\_runtime\_resolve，为动态链接器的入口。

7 . 当动态链接器获得控制之后，其展开栈，查看指定的重定位项，发现符号的值，将name1的“实际”地址存放在它的全局偏移表项中，然后将控制转换到所希望的目的地。

0x804a008 :   0x0804831e

让我们看看dynamic linker如何修改这个，我们在0x804a008上设置一个硬件watchpoint

(gdb) watch \*0x804a008
Hardware watchpoint 2: \*0x804a008
(gdb) c
Continuing.
Hardware watchpoint 2: \*0x804a008

Old value = 134513438
New value = 1616016
\_dl\_fixup (l=, reloc\_arg=) at dl-runtime.c:155
155     dl-runtime.c: No such file or directory.
       in dl-runtime.c

我们可以看到，地址0x804a008上的内容，从134513438 变化到了1616016，

(gdb) p/x 134513438
$2 = 0x804831e
(gdb) p/x 1616016
$3 = 0x18a890

我们看看 这个新地址 (1616016 0x18a890) 是什么

(gdb) disassemble 0x18a890,0x18a8a0
Dump of assembler code from 0x18a890 to 0x18a8a0:
  0x0018a890 :     push   %ebp
  0x0018a891 :     mov    %esp,%ebp
  0x0018a893 :     sub    $0x20,%esp
  0x0018a896 :     mov    %ebx,-0xc(%ebp)
  0x0018a899 :     mov    0x8(%ebp),%eax
  0x0018a89c :    call   0x143a0f

Yay!, 我们能看到地址0x804a008上的内容已经变化成为了实际的glibc中的地址了。

(gdb) bt
#0  \_dl\_fixup (l=, reloc\_arg=) at dl-runtime.c:155
#1  0x00123280 in \_dl\_runtime\_resolve () at ../sysdeps/i386/dl-trampoline.S:37
#2  0x080483f9 in main () at plt.c:6
