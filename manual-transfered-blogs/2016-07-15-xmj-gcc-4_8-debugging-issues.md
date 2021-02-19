转换时间： 2021-02-01

# GCC 4.8之后的调试问题
Posted on 2016/07/15 by xmj	

gdb无法调试4.8或者更新版本gcc编译的程序，这个问题比较常见。原因很简单，参见 https://gcc.gnu.org/gcc-4.8/changes.html ：
```
DWARF4 is now the default when generating DWARF debug information. When -g is used on a platform that uses DWARF debugging information, GCC will now default to -gdwarf-4 -fno-debug-types-section.
GDB 7.5, Valgrind 3.8.0 and elfutils 0.154 debug information consumers support DWARF4 by default. Before GCC 4.8 the default version used was DWARF2. To make GCC 4.8 generate an older DWARF version use -g together with -gdwarf-2 or -gdwarf-3. The default for Darwin and VxWorks is still -gdwarf-2 -gstrict-dwarf.
```
这是因为，gcc 4.8缺省生成dwarf4版本的调试信息，旧版本的gdb无法识别。所以，要么升级gdb，要么通过选项-g -gdwarf-2或者-gdwarf-3来生成低版本的dwarf调试信息。

注：查看ELF文件的dwarf版本信息，可以使用如下命令
```
readelf -wi a.out | grep Version
```
