转换时间： 2021-01-31

# 有趣的选项 —— ld –wrap=symbol
Posted on 2011/12/15 by xmj

工具链的选项非常丰富，有许多功能其实都可以通过选项来实现，比如如下ld的一个选项，我直接把手册的内容贴了下来。里面介绍了如何通过该选项，实现对系统函数进行封装，并且列举了一个例子。
```
–wrap=symbol
Use a wrapper function for symbol. Any undefined reference to symbol will be resolved to __wrap_symbol. Any undefined reference to __real_symbol will be resolved to symbol.

This can be used to provide a wrapper for a system function. The wrapper function should be called __wrap_symbol. If it wishes to call the system function, it should call __real_symbol.

Here is a trivial example:

              void *
              __wrap_malloc (size_t c)
              {
                printf ("malloc called with %zu\n", c);
                return __real_malloc (c);
              }

If you link other code with this file using –wrap malloc, then all calls to malloc will call the function __wrap_malloc instead. The call to __real_malloc in __wrap_malloc will call the real malloc function.

You may wish to provide a __real_malloc function as well, so that links without the –wrap option will succeed. If you do this, you should not put the definition of __real_malloc in the same file as __wrap_malloc; if you do, the assembler may resolve the call before the linker has a chance to wrap it to malloc.
```
