---
title: "gcc的 \"-fpack-struct\" 编译选项导致程序core dump的分析"
date: "2014-09-26"
categories: 
  - "gcc"
---

最近team引入`gcov`来做代码分析。编译好的程序在`Solaris`上运行的好好的，结果在`Linux`上一运行就会产生`core dump`文件。这篇文章就介绍整个分析过程。

首先用`gdb`分析`core`文件,显示是`strlen`调用出了问题：

```
(gdb) bt
#0  0x00000034e433386f in __strlen_sse42 () from /lib64/libc.so.6
#1  0x000000000053c57a in __gcov_init ()
#2  0x000000000053c4b9 in _GLOBAL__I_65535_0_g_cmd_param () at source/rerun/aicent_ara_rerun.c:963
#3  0x000000000053dc26 in __do_global_ctors_aux ()
#4  0x0000000000403743 in _init ()
#5  0x00007fff6d6b3ce8 in ?? ()
#6  0x000000000053db55 in __libc_csu_init ()
#7  0x00000034e421ecb0 in __libc_start_main () from /lib64/libc.so.6
#8  0x0000000000404449 in _start ()
```

由于我们使用的`gcc`是用安装包形式安装的，没有源码。所以就从`github`上找了相应版本的`gcc`源代码，希望能有所帮助。以下是`__gcov_init`函数的代码（[https://github.com/gcc-mirror/gcc/blob/gcc-4\_4\_7-release/gcc/libgcov.c](https://github.com/gcc-mirror/gcc/blob/gcc-4_4_7-release/gcc/libgcov.c)）：

```
void
__gcov_init (struct gcov_info *info)
{
  if (!info->version)
    return;
  if (gcov_version (info, info->version, 0))
    {
      const char *ptr = info->filename;
      gcov_unsigned_t crc32 = gcov_crc32;
      size_t filename_length =  strlen(info->filename);

      /* Refresh the longest file name information */
      if (filename_length > gcov_max_filename)
        gcov_max_filename = filename_length;

      do
    {
      unsigned ix;
      gcov_unsigned_t value = *ptr << 24;

      for (ix = 8; ix--; value <<= 1)
        {
          gcov_unsigned_t feedback;

          feedback = (value ^ crc32) & 0x80000000 ? 0x04c11db7 : 0;
          crc32 <<= 1;
          crc32 ^= feedback;
        }
    }
      while (*ptr++);

      gcov_crc32 = crc32;

      if (!gcov_list)
    atexit (gcov_exit);

      info->next = gcov_list;
      gcov_list = info;
    }
  info->version = 0;
}
```

结合源代码和`core`文件可以看出，应该是“`size_t filename_length = strlen(info->filename);`”这一行出了问题。再结合汇编程序：

```
(gdb) disassemble __strlen_sse42
Dump of assembler code for function __strlen_sse42:
   0x00000034e4333860 <+0>:     pxor   %xmm1,%xmm1
   0x00000034e4333864 <+4>:     mov    %edi,%ecx
   0x00000034e4333866 <+6>:     mov    %rdi,%r8
   0x00000034e4333869 <+9>:     and    $0xfffffffffffffff0,%rdi
   0x00000034e433386d <+13>:    xor    %edi,%ecx
=> 0x00000034e433386f <+15>:    pcmpeqb (%rdi),%xmm1
```

是访问`rdi`寄存器出了问题，而`rdi`保存的应该是`strlen`的参数，也就是“`info->filename`”。试着访问一下`rdi`寄存器保存的地址：

```
(gdb) i registers rdi
rdi            0x57c4ac00000000 24704565987246080
(gdb) x/16xb 0x57c4ac00000000
0x57c4ac00000000:       Cannot access memory at address 0x57c4ac00000000
```

可以看到`rdi`寄存器保存的地址的确是个无效的地址。

接下来，就要分析一下为什么传入`__gcov_init`的`info`结构体的`filename`是一个无效指针。首先看一下`gcov_info`结构体的定义（[https://github.com/gcc-mirror/gcc/blob/gcc-4\_4\_7-release/gcc/gcov-io.h](https://github.com/gcc-mirror/gcc/blob/gcc-4_4_7-release/gcc/gcov-io.h)）：

```
/* Information about a single object file.  */
struct gcov_info
{
  gcov_unsigned_t version;  /* expected version number */
  struct gcov_info *next;   /* link to next, used by libgcov */

  gcov_unsigned_t stamp;    /* uniquifying time stamp */
  const char *filename;     /* output file name */

  unsigned n_functions;     /* number of functions */
  const struct gcov_fn_info *functions; /* table of functions */

  unsigned ctr_mask;        /* mask of counters instrumented.  */
  struct gcov_ctr_info counts[0]; /* count data. The number of bits
                     set in the ctr_mask field
                     determines how big this array
                     is.  */
};
```

查看调用`__gcov_init`的`_GLOBAL__I_65535_0_g_cmd_param`函数的汇编代码：

```
(gdb) disassemble _GLOBAL__I_65535_0_g_cmd_param
Dump of assembler code for function _GLOBAL__I_65535_0_g_cmd_param:
   0x000000000053c4ab <+0>:     push   %rbp
   0x000000000053c4ac <+1>:     mov    %rsp,%rbp
   0x000000000053c4af <+4>:     mov    $0x78d4a0,%edi
   0x000000000053c4b4 <+9>:     callq  0x53c4c0 <__gcov_init>
   0x000000000053c4b9 <+14>:    leaveq
   0x000000000053c4ba <+15>:    retq
End of assembler dump.
```

可以看到传入`__gcov_init`的参数为`0x78d4a0`，也就是指向`gcov_info`结构体的地址，查看这个地址的内容：

```
(gdb) x/64xb 0x78d4a0
0x78d4a0:       0x52    0x34    0x30    0x34    0x00    0x00    0x00    0x00
0x78d4a8:       0x00    0x00    0x00    0x00    0x82    0xf0    0xc7    0xa5
0x78d4b0:       0x60    0xc4    0x57    0x00    0x00    0x00    0x00    0x00
0x78d4b8:       0x0b    0x00    0x00    0x00    0xac    0xc4    0x57    0x00
0x78d4c0:       0x00    0x00    0x00    0x00    0x01    0x00    0x00    0x00
0x78d4c8:       0x39    0x01    0x00    0x00    0xc0    0xa4    0x47    0x03
0x78d4d0:       0x00    0x00    0x00    0x00    0xe0    0xd8    0x53    0x00
0x78d4d8:       0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00
```

可以看到对应`filename`成员的值应该为`0x57c4ac0000000b`，的确是个无效地址。问题分析到这里，没了思路。后来，在`gcc`的`bugzilla`里找到这个问题：[https://gcc.gnu.org/bugzilla/show\_bug.cgi?id=43341](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=43341)，才搞清楚是“`-fpack-struct=4`”这个编译选项导致的。

我们使用的是64位Linux，默认编译生成的可执行文件是64位的。所以`gcov_info`的默认内存布局应该是（`gcov_unsigned_t`类型占4个字节，指针类型占8个字节）：

| Offset | 4 bytes | 4 bytes |
| --- | --- | --- |
| 0 | version | 填充成员 |
| 8 | next | next |
| 16 | stamp | 填充成员 |
| 24 | filename | filename |
| … | … | … |

当使用“`-fpack-struct=4`”这个编译选项后，`gcov_info`的内存布局变为：

| Offset | 4 bytes | 4 bytes |
| --- | --- | --- |
| 0 | version | next |
| 8 | next | stamp |
| 16 | filename | filename |
| … | … | … |

经过推算，`filename`成员的值应该为`0x57c460`，验证一下：

```
(gdb) p (char*)0x57c460
$1 = 0x57c460 "/home/.../.....gcda"
```

打印出的是正确的值。在`Solaris`上没问题的原因是因为64位`Solaris`默认编译出来的程序是32位的。

看了一下`gcc`网站对`-fpack-struct`的介绍（[https://gcc.gnu.org/onlinedocs/gcc/Code-Gen-Options.html#index-fpack-struct-2675](https://gcc.gnu.org/onlinedocs/gcc/Code-Gen-Options.html#index-fpack-struct-2675)），使用这个编译选项会导致`ABI(Application Binary Interface)`的改变，所以使用时一定要谨慎。
