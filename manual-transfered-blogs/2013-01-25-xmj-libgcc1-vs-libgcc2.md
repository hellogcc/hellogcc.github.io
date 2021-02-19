转换时间： 2021-01-31

# libgcc1和libgcc2的区别
Posted on 2013/01/25 by xmj	

参见：http://gcc.gnu.org/onlinedocs/gccint/Libgcc.html#Libgcc

GCC provides a low-level runtime library, libgcc.a or libgcc_s.so.1 on some platforms. GCC generates calls to routines in this library automatically, whenever it needs to perform some operation that is too complicated to emit inline code for.

Most of the routines in libgcc handle arithmetic operations that the target processor cannot perform directly. This includes integer multiply and divide on some machines, and all floating-point and fixed-point operations on other machines. libgcc also includes routines for exception handling, and a handful of miscellaneous operations.

libgcc是GCC提供的一个低层运行时库，当一些操作/运算在特定平台上不支持时，GCC会自动生成对这些库函数的调用，使用这些库函数来模拟实现。从概念上和源码实现中，又可以分为libgcc1和libgcc2，虽然它们最终会被编译合并为libgcc.a。

参见：http://gcc.gnu.org/ml/gcc-help/2009-03/msg00145.html

libgcc1 exists primarily conceptually. It is the basic set of
operations which can not be reasonably implemented using other
operations. In the good old days libgcc1 was built using the other
compiler on your system. Since these days there is generally no other
compiler, most targets provide assembler code to perform the operations.
For example, see config/arm/lib1funcs.asm.

libgcc1中包含了一套基础操作/运算，这些无法使用其它操作来实现，通常会使用一系列的汇编代码来模拟完成。

libgcc2, conversely, is the set of operations which can be implemented
reasonably. For example, if you have a 32-bit add instruction, it’s
easy to use it to implement 64-bit addition. This code appears in
gcc/libgcc2.c. On many processors it is possible to optimize using
instructions which are not avaliable in C, such as add-with-carry; those
optimizations are written in gcc/longlong.h.

libgcc2，正好相反，这些可以通过已有的一些操作/运算来简单的组合完成。通常是使用C代码来编写。

另参见：http://gcc.gnu.org/ml/gcc/2003-12/msg01191.html
