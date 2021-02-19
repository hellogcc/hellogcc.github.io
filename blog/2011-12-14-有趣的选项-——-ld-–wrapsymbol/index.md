---
title: "有趣的选项 —— ld –wrap=symbol"
date: "2011-12-14"
categories: 
  - "其它技术"
tags: 
  - "ld-linker"
---

工具链的选项非常丰富，有许多功能其实都可以通过选项来实现，比如如下ld的一个选项，我直接把手册的内容贴了下来。里面介绍了如何通过该选项，实现对系统函数进行封装，并且列举了一个例子。

–wrap=symbol  
Use a wrapper function for symbol. Any undefined reference to symbol will be resolved to \_\_wrap\_symbol. Any undefined reference to \_\_real\_symbol will be resolved to symbol.

This can be used to provide a wrapper for a system function. The wrapper function should be called \_\_wrap\_symbol. If it wishes to call the system function, it should call \_\_real\_symbol.

Here is a trivial example:

              void \*
              \_\_wrap\_malloc (size\_t c)
              {
                printf ("malloc called with %zu\\n", c);
                return \_\_real\_malloc (c);
              }

If you link other code with this file using –wrap malloc, then all calls to malloc will call the function \_\_wrap\_malloc instead. The call to \_\_real\_malloc in \_\_wrap\_malloc will call the real malloc function.

You may wish to provide a \_\_real\_malloc function as well, so that links without the –wrap option will succeed. If you do this, you should not put the definition of \_\_real\_malloc in the same file as \_\_wrap\_malloc; if you do, the assembler may resolve the call before the linker has a chance to wrap it to malloc.
