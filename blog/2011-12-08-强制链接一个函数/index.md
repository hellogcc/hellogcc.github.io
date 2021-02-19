---
title: "强制链接一个函数"
date: "2011-12-08"
categories: 
  - "gcc"
---

xmj@hellogcc

1、有时候，我们在程序里定义了一个函数，但是没有显式的调用它，只是用于其它目的比如方便调试。我们不想让编译器将它优化掉。这个时候，可以使用GCC扩展语法，来指定该函数需要保留。这在GCC源代码中也被用到，例如：

#if (GCC\_VERSION > 4000)
#define DEBUG\_FUNCTION \_\_attribute\_\_ ((\_\_used\_\_))
#define DEBUG\_VARIABLE \_\_attribute\_\_ ((\_\_used\_\_))
#else 
#define DEBUG\_FUNCTION
#define DEBUG\_VARIABLE
#endif

DEBUG\_FUNCTION void
debug\_bb (basic\_block bb)
{
  dump\_bb (bb, stderr, 0);
}

2、但是，如果这个函数是在库中，而我们仍然希望将其链接到应用程序中的话，上述方法就不起作用了。这个时候，则可以通过链接器参数来指定。例如，

$ gcc foo.c -Wl,-uprintf -lc

\-u的作用是指定该符合，printf，未定义，从而强制将其链接到程序中。
