转换时间： 2021-02-01

# 如何写gdb命令脚本
Posted on 2014/07/02 by nanxiao

作为UNIX/Linux下使用广泛的调试器，gdb不仅提供了丰富的命令，还引入了对脚本的支持：一种是对已存在的脚本语言支持，比如python，用户可以直接书写python脚本，由gdb调用python解释器执行；另一种是命令脚本（command file），用户可以在脚本中书写gdb已经提供的或者自定义的gdb命令，再由gdb执行。在这篇文章里，我会介绍一下如何写gdb的命令脚本。

(一) 自定义命令
gdb支持用户自定义命令，格式是：
```
define commandName  
    statement  
    ......  
end  
```
其中statement可以是任意gdb命令。此外自定义命令还支持最多10个输入参数：$arg0，$arg1 …… $arg9，并且还用$argc来标明一共传入了多少参数。

下面结合一个简单的C程序（test.c），来介绍如何写自定义命令：
```
#include <stdio.h>

int global = 0;

int fun_1(void)
{
    return 1;
}

int fun_a(void)
{
    int a = 0;
    printf("%d\n", a);
}

int main(void)
{
    fun_a();
    return 0;
}
```
首先编译成可执行文件：
```
gcc -g -o test test.c
```
接着用gdb进行调试：
```
[root@linux:~]$ gdb test
GNU gdb (GDB) 7.6
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-unknown-linux-gnu".
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>...
Reading symbols from /data2/home/nanxiao/test...done.
(gdb) b fun_a
Breakpoint 1 at 0x4004d7: file test.c, line 12.
(gdb) r
Starting program: /data2/home/nanxiao/test

Breakpoint 1, fun_a () at test.c:12
12              int a = 0;
(gdb) bt
#0  fun_a () at test.c:12
#1  0x0000000000400500 in main () at test.c:18
```
可以看到使用bt（backtrace缩写）命令可以打印当前线程的调用栈。我们的第一个自定义命令就是也实现一个backtrace功能：
```
define mybacktrace
    bt
end
```
怎么样？简单吧，纯粹复用gdb提供的命令。下面来验证一下：
```
(gdb) define mybacktrace
Type commands for definition of "mybacktrace".
End with a line saying just "end".
>bt
>end
(gdb) mybacktrace
#0  fun_a () at test.c:12
#1  0x0000000000400500 in main () at test.c:18
```
功能完全正确！

接下来定义一个赋值命令，把第二个参数的值赋给第一个参数：
```
define myassign
    set var $arg0 = $arg1
end
```
执行一下：
```
(gdb) define myassign
Type commands for definition of "myassign".
End with a line saying just "end".
>set var $arg0 = $arg1
>end
(gdb) myassign global 3
(gdb) p global
$1 = 3
```
可以看到global变量的值变成了3。

对于自定义命令来说，传进来的参数只是进行简单的文本替换，所以你可以传入赋值的表达式，甚至是函数调用：
```
(gdb) myassign global fun_1()
(gdb) p global
$2 = 1
```
可以看到global变量的值变成了1。

除此以外，还可以为自定义命令写帮助文档，也就是执行help命令时打印出的信息：
```
document myassign
    assign the second parameter value to the first parameter
end
```
执行help命令：
```
(gdb) document myassign
Type documentation for "myassign".
End with a line saying just "end".
>assign the second parameter value to the first parameter
>end
(gdb) help myassign
assign the second parameter value to the first parameter
```
可以看到打印出了myassign的帮助信息。

（二） 命令脚本
首先对于命令脚本的命名，其实gdb没有什么特殊要求，只要文件名不是gdb支持的其它脚本语言的文件名就可以了（比如.py）。因为这样做会使gdb按照相应的脚本语言去解析命令脚本，结果自然是不对的。

其次为了帮助用户写出功能强大的脚本，gdb提供了如下的流程控制命令：
（1）条件命令：if...else...end。这个同其它语言中提供的if命令没什么区别，只是注意结尾的end。
（2）循环命令：while...end。gdb同样提供了loop_break和loop_continue命令分别对应其它语言中的break和continue，另外同样注意结尾的end。

另外gdb还提供了很多输出命令。比方说echo命令，如果仅仅是输出一段文本，echo命令特别方便。此外还有和C语言很相似的支持格式化输出的printf命令，等等。

脚本文件的注释也是以#开头的，这个同很多其它脚本语言都一样。

最后指出的是在gdb中执行脚本要使用source命令，例如：“source xxx.gdb”。

（三） 一个完整的例子

最后以一个完整的gdb脚本（search_byte.gdb）做例子，来总结一下本文提到的内容:
```
define search_byte
    if $argc != 3
        help search_byte
    else
        set $begin_addr = $arg0
        set $end_addr = $arg1

        while $begin_addr <= $end_addr
            if *((unsigned char*)$begin_addr) == $arg2
                printf "Find it！The address is 0x%x\n", $begin_addr
                loop_break
            else
                set $begin_addr = $begin_addr + 1
            end
        end

        if $begin_addr > $end_addr
            printf "Can't find it!\n"
        end
    end
end

document search_byte
    search a specified byte value(0 ~ 255) during a memory
    usage: search_byte begin_addr end_addr byte
end
```
这个脚本定义了search_byte命令，目的是在一段指定内存查找一个值（unsigned char类型）：需要输入内存的起始地址，结束地址和要找的值。
命令逻辑可以分成3个部分：
（a） 首先判断输入参数是不是3个，如果不是，就输出帮助信息；
（b） 从起始地址开始查找指定的值，如果找到，打印地址值并退出循环，否则把地址加1，继续查找；
（c） 如果在指定内存区域没有找到，打印提示信息。

另外这个脚本还定义了search_byte的帮助信息。

仍然以上面的C程序为例，来看一下如何使用这个gdb脚本：
```
[root@linux:~]$ gdb test
GNU gdb (GDB) 7.6
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-unknown-linux-gnu".
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>...
Reading symbols from /data2/home/nanxiao/test...done.
(gdb) p &global
$1 = (int *) 0x600900 <global>
(gdb) p global
$2 = 0
(gdb) source search_byte.gdb
(gdb) search_byte 0x600900 0x600903 0
Find it! The address is 0x600900
(gdb) search_byte 0x600900 0x600903 1
Can't find it!
```
可以看到global的值是0，起始地址是0x600900，结束地址是0x600903。在global的内存区域查找0成功，查找1失败。

受篇幅所限，本文只是对gdb命令脚本做了一个粗浅的介绍，旨在起到抛砖引玉的效果。如果大家想更深入地了解这部分知识，可以参考gdb手册的相关章节：Extending GDB (https://sourceware.org/gdb/onlinedocs/gdb/Extending-GDB.html)。最后向大家推荐一个github上的.gdbinit文件：https://github.com/gdbinit/Gdbinit/blob/master/gdbinit，把这个弄懂，相信gdb脚本文件就不在话下了。

参考文献：
（1）Extending GDB (https://sourceware.org/gdb/onlinedocs/gdb/Extending-GDB.html);
（2）捉虫日记（http://www.ituring.com.cn/book/909）。
