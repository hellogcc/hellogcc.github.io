---
title: "小技巧：使用预处理选项-P"
date: "2011-09-24"
categories: 
  - "gcc"
---

xmj@hellogcc.org

有时编译程序会遇到如下所类似的错误，  
`  
In file included from foo.c:15,  
from a.h:45,  
b.h:53: error: ... ...  
`  
为了查看出错原因，可能会需要先手动编译生成相应的预处理文件，然后查看预处理文件中的代码。比如，对于  
`  
gcc -o foo.o -c foo.c  
`  
可以先运行  
`  
gcc -o foo.i -E foo.c  
`  
来生成foo.i预处理文件。然后，还可以尝试手动编译，修改这个预处理文件。

但是，由于生成的预处理文件中含有行号标记（linemarker），所以，运行  
`  
gcc -o foo.o -c foo.i  
`  
所得到的错误信息还是跟最初的一样，如果可以将预处理文件中的行号标记都去掉，似乎会有些帮助。

幸好，gcc提供了这个选项：  
`  
-P  
Inhibit generation of linemarkers in the output from the  
preprocessor. This might be useful when running the preprocessor on  
something that is not C code, and will be sent to a program which  
might be confused by the linemarkers.  
`  
运行  
`  
gcc -o foo.i -E foo.c -P  
`  
即可。
