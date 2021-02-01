转换时间： 2021-02-01

# gcc的 “-fpack-struct” 编译选项导致程序core dump的分析
Posted on 2014/09/26 by nanxiao

最近team引入gcov来做代码分析。编译好的程序在Solaris上运行的好好的，结果在Linux上一运行就会产生core dump文件。这篇文章就介绍整个分析过程。

首先用gdb分析core文件,显示是strlen调用出了问题：
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
由于我们使用的gcc是用安装包形式安装的，没有源码。所以就从github上找了相应版本的gcc源代码，希望能有所帮助。以下是__gcov_init函数的代码（https://github.com/gcc-mirror/gcc/blob/gcc-4_4_7-release/gcc/libgcov.c）：
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
结合源代码和core文件可以看出，应该是“size_t filename_length = strlen(info->filename);”这一行出了问题。再结合汇编程序：
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
是访问rdi寄存器出了问题，而rdi保存的应该是strlen的参数，也就是“info->filename”。试着访问一下rdi寄存器保存的地址：
```
(gdb) i registers rdi
rdi            0x57c4ac00000000 24704565987246080
(gdb) x/16xb 0x57c4ac00000000
0x57c4ac00000000:       Cannot access memory at address 0x57c4ac00000000
```
可以看到rdi寄存器保存的地址的确是个无效的地址。

接下来，就要分析一下为什么传入__gcov_init的info结构体的filename是一个无效指针。首先看一下gcov_info结构体的定义（https://github.com/gcc-mirror/gcc/blob/gcc-4_4_7-release/gcc/gcov-io.h）：
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
查看调用__gcov_init的_GLOBAL__I_65535_0_g_cmd_param函数的汇编代码：
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
可以看到传入__gcov_init的参数为0x78d4a0，也就是指向gcov_info结构体的地址，查看这个地址的内容：
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
可以看到对应filename成员的值应该为0x57c4ac0000000b，的确是个无效地址。问题分析到这里，没了思路。后来，在gcc的bugzilla里找到这个问题：https://gcc.gnu.org/bugzilla/show_bug.cgi?id=43341，才搞清楚是“-fpack-struct=4”这个编译选项导致的。

我们使用的是64位Linux，默认编译生成的可执行文件是64位的。所以gcov_info的默认内存布局应该是（gcov_unsigned_t类型占4个字节，指针类型占8个字节）：

```
Offset 	4 bytes 	4 bytes
0 	version 	填充成员
8 	next 	next
16 	stamp 	填充成员
24 	filename 	filename
… 	… 	…
```
当使用“-fpack-struct=4”这个编译选项后，gcov_info的内存布局变为：
```
Offset 	4 bytes 	4 bytes
0 	version 	next
8 	next 	stamp
16 	filename 	filename
… 	… 	…
```
经过推算，filename成员的值应该为0x57c460，验证一下：
```
(gdb) p (char*)0x57c460
$1 = 0x57c460 "/home/.../.....gcda"
```
打印出的是正确的值。在Solaris上没问题的原因是因为64位Solaris默认编译出来的程序是32位的。

看了一下gcc网站对-fpack-struct的介绍（https://gcc.gnu.org/onlinedocs/gcc/Code-Gen-Options.html#index-fpack-struct-2675），使用这个编译选项会导致ABI(Application Binary Interface)的改变，所以使用时一定要谨慎。
