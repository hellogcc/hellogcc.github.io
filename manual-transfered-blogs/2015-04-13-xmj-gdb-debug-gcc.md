转换时间： 2021-02-01

# gdb无法调试最新gcc编译的程序
Posted on 2015/04/13 by xmj
```
Reading symbols from /home/foo/helloword...done.
(gdb) start
Temporary breakpoint 1 at 0x4004dc
Starting program: /home/foo/helloword

Temporary breakpoint 1, 0x00000000004004dc in main ()
(gdb) n
Single stepping until exit from function main,
which has no line number information.
Hello World
__libc_start_main (main=0x4004d8
, argc=1, ubp_av=0x7fffffffc2b8, init=, fini=,
    rtld_fini=, stack_end=0x7fffffffc2a8) at libc-start.c:258
258	  exit (result);
(gdb)
```
这里，gcc是较新的版本，gdb版本较旧。原因是因为，gcc从4.8开始缺省使用了-gdwarf-4选项，较旧的gdb无法识别dwarf4版本的调试信息（多谢jasonwucj的指点）。参见

https://gcc.gnu.org/gcc-4.8/changes.html

当无法更新gdb的时候，则可以在用gcc编译程序时，使用选项-gdwarf-3来指定生成dwarf3版本的调试信息，这样就可以了。
