转换时间：
2021-01-31

# 强制链接一个函数
Posted on 2011/12/08 by xmj

xmj@hellogcc

1、有时候，我们在程序里定义了一个函数，但是没有显式的调用它，只是用于其它目的比如方便调试。我们不想让编译器将它优化掉。这个时候，可以使用GCC扩展语法，来指定该函数需要保留。这在GCC源代码中也被用到，例如：
```
#if (GCC_VERSION > 4000)
#define DEBUG_FUNCTION __attribute__ ((__used__))
#define DEBUG_VARIABLE __attribute__ ((__used__))
#else
#define DEBUG_FUNCTION
#define DEBUG_VARIABLE
#endif
```

```
DEBUG_FUNCTION void
debug_bb (basic_block bb)
{
  dump_bb (bb, stderr, 0);
}
```
2、但是，如果这个函数是在库中，而我们仍然希望将其链接到应用程序中的话，上述方法就不起作用了。这个时候，则可以通过链接器参数来指定。例如，
```
$ gcc foo.c -Wl,-uprintf -lc
```
`-u` 的作用是指定该符合，printf，未定义，从而强制将其链接到程序中。
