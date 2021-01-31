转换时间： 2021-01-31

# 使用gdb pretty printer调试go dist
Posted on 2013/09/29 by xmj	

作者：xmj

1、GDB提供了Pretty Printer功能，可以在调试程序的时候，通过自定义的打印程序来更好的显示一个数据结构的值。

相关信息可以参考gdb手册：
23.2.2.7 Writing a Pretty-Printer
https://sourceware.org/gdb/current/onlinedocs/gdb/Writing-a-Pretty_002dPrinter.html#Writing-a-Pretty_002dPrinter

2、Go源码包里的dist目录，go/src/cmd/dist，是Go的bootstraping工具，用来自举go编译器，以及各种可以使用脚本来完成的杂活。其本身是用C写的，里面有两个常用的数据类型，定义在a.h里：
```
// A Buf is a byte buffer, like Go's []byte.
typedef struct Buf Buf;
struct Buf
{
        char *p;
        int len;
        int cap;
};

// A Vec is a string vector, like Go's []string.
typedef struct Vec Vec;
struct Vec
{
        char **p;
        int len;
        int cap;
};
```
我们在gdb中调试dist的时候，如果想查看一个Buf或者Vec的内容，如下：
```
(gdb) p b
$1 = {p = 0x80574b8, len = 0x3, cap = 0x40}
(gdb) p gccargs
$2 = {p = 0x8059778, len = 0x4, cap = 0x40}
```
显然，不是很令人满意。

3、为此，我们可以为其定义pretty printers。这里，需要两部分内容，首先创建一个目录，以及一个文件：python/printers.py，内容如下：
```
import gdb

class BufPrinter(object):
    "Print a struct Buf."

    def __init__ (self, val):
        self.val = val

    def to_string (self):
        p = self.val['p']
        l = self.val['len']
        ret = "\""
        ret += p.string(length = l)
        ret += "\""
        return ret

class VecPrinter(object):
    "Print a struct Vec."

    def __init__ (self, val):
        self.val = val

    def to_string (self):
        p = self.val['p']
        l = self.val['len']
        ret = "\""
        for i in range(l):
            if i > 0:
                ret += " "
            ret += p[i].string()
        ret += "\""
        return ret

def build_pretty_printer():
    pp = gdb.printing.RegexpCollectionPrettyPrinter("golang")
    pp.add_printer('Buf', '^Buf$', BufPrinter)
    pp.add_printer('Vec', '^Vec$', VecPrinter)
    return pp

def register_golang_printer():
    gdb.printing.register_pretty_printer(
        gdb.current_objfile(),
        build_pretty_printer())
```
然后，在~/.gdbinit里，增加如下代码：
```
python
import sys
sys.path.insert(0, '/path/to/your/python')
from printers import register_golang_printer
register_golang_printer ()
end
```
4、此时，再通过gdb调试dist，显示如下：
```
(gdb) p b
$1 = "gcc"
(gdb) p gccargs
$2 = "gcc -Wall -Wstrict-prototypes -Wextra"
```
