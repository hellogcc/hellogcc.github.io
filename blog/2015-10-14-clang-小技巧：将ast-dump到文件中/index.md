---
title: "[clang] 小技巧：将AST Dump到文件中"
date: "2015-10-14"
categories: 
  - "clang"
---

使用如下命令，可以将AST打印到屏幕上： `clang -Xclang -ast-dump test.c`

如果要将打印内容保存到文件中，可以使用如下命令： `TERM="" clang -Xclang -ast-dump test.c > ast.dump`
