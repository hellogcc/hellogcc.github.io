转换时间： 2021-02-01

# 在Solaris上使用LD_OPTIONS环境变量诊断编译链接问题
Posted on 2014/10/10 by nanxiao	

最近在Solaris上编译一款开源软件，在最后链接阶段出了问题，导致ld程序core dump。由于没有ld程序源代码，导致完全没思路，没办法，只好在stackoverflow上求教：http://stackoverflow.com/questions/26009192/why-the-ld-crash-in-building-libgd。从回复中我才知道可以通过设置LD_OPTIONS环境变量，来了解整个链接过程。举个例子：

LD_OPTIONS=-Dfiles,detail

指定files会输出ld当前处理的文件，detail会提供更多的信息。

可以用“ld -Dhelp”命令打印每个选项的详细帮助信息。

如果想详细了解Solaris下程序的链接过程，可以参考这篇文档：Linker and Libraries Guide。
http://docs.oracle.com/cd/E19683-01/817-3677/817-3677.pdf
