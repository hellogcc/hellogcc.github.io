转换时间： 2021-01-31

# ?: in tc/expect
Posted on 2012/09/11 by yao

今天看GDB testsuite里边的一个正则表达匹配，里边出现了 ?:，

set breakpoint_re "=(?:breakpoint-created|breakpoint-deleted)\[^\n\]+\"\r\n"

看了好久都不明白，google了半天，才知道。在tcl/expect 中，() 可以捕获里边匹配的内容，比如 (http|ftp)捕获URL里边的协议类型，但是 ?:的作用是匹配括号里边的正则表达式，但是不捕获这些内容。详细参考这里 http://www.tcl.tk/doc/howto/regexp81.tml
