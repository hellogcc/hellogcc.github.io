---
title: "clang小技巧：加快编译速度"
date: "2015-03-18"
categories: 
  - "clang"
  - "llvm"
---

编译整个llvm会花费很长的时间，如果你只关心clang，则每次修改完代码之后需要make的时候，可以使用

`make ONLY_TOOLS="clang"`

只编译clang，从而大大减少编译时间。

进一步，还可以使用

`make ONLY_TOOLS="clang" BUILD_CLANG_ONLY=YES`

只编译器clang本身，不编译clang的unit test。
