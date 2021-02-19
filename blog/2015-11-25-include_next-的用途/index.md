---
title: "'#include_next' 的用途"
date: "2015-11-25"
categories: 
  - "gcc"
tags: 
  - "gcc"
  - "gnu-extension"
---

最近在阅读开源项目代码的时候看到一个很少见的预处理命令:

`#include_next`

查询了一下, 并不是标准中的一部分, 属于 GNU 扩展, 使用的场合也比较少, 在某些新旧代码共存时或许会比较常见.

[https://gcc.gnu.org/onlinedocs/cpp/Wrapper-Headers.html](https://gcc.gnu.org/onlinedocs/cpp/Wrapper-Headers.html)

> 2.7 Wrapper Headers
> 
> Sometimes it is necessary to adjust the contents of a system-provided header file without editing it directly. GCC's fixincludes operation does this, for example. One way to do that would be to create a new header file with the same name and insert it in the search path before the original header. That works fine as long as you're willing to replace the old header entirely. But what if you want to refer to the old header from the new one?
> 
> You cannot simply include the old header with ‘#include’. That will start from the beginning, and find your new header again. If your header is not protected from multiple inclusion (see Once-Only Headers), it will recurse infinitely and cause a fatal error.
> 
> You could include the old header with an absolute pathname:
> 
> #include "/usr/include/old-header.h"
> 
> This works, but is not clean; should the system headers ever move, you would have to edit the new headers to match.
> 
> There is no way to solve this problem within the C standard, but you can use the GNU extension ‘#include\_next’. It means, “Include the next file with this name”. This directive works like ‘#include’ except in searching for the specified file: it starts searching the list of header file directories after the directory in which the current file was found.
