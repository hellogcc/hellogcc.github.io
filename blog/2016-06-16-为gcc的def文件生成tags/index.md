---
title: "为gcc源代码生成tags"
date: "2016-06-16"
categories: 
  - "gcc"
---

LLVM的td文件使用的是tablegen语言，所以，它可以增加一个tablegen后端，来输出td文件的tags。

gcc的def文件使用的是宏定义，格式比较随意，很难通过类似llvm这样的方式，来为def文件生成tags。好在，大多def文件都是这样的一个格式:

DEF (ENUM, NAME, ...)

例如：

DEF\_GOMP\_BUILTIN (BUILT\_IN\_OMP\_GET\_NUM\_THREADS, "omp\_get\_num\_threads",
                  BT\_FN\_INT, ATTR\_CONST\_NOTHROW\_LEAF\_LIST)

通过如下的ctags命令，可以粗略的为这些def文件生成tags索引：

$ ctags --langdef=def --langmap=def:.def --regex-def="/^\[ \\t\]\*\[a-zA-Z0-9\_\]+\[ \\t\]\*\\(\[ \\t\]\*(\[a-zA-Z0-9\_\]+)/\\1/d,definition/" -R .

可以看到tags文件里生成了如下内容：

BUILT\_IN\_OMP\_GET\_NUM\_THREADS    omp-builtins.def        /^DEF\_GOMP\_BUILTIN (BUILT\_IN\_OMP\_GET\_NUM\_THREADS, "omp\_get\_num\_threads",$/;"    d

目前gcc的代码虽然是c++实现，但是后缀名还是“.c”。所以，需要加上一个额外选项告诉ctags：

\--langmap=c++:+.c

去掉testsuite目录:

\--exclude=testsuite

过滤掉代码中一些宏的干扰：

\-I GTY+ -I VEC+ -I DEF\_VEC\_P+

把build目录加进来，合起来的命令如下：

$ ctags --langdef=def --langmap=def:.def --regex-def="/^\[ \\t\]\*\[a-zA-Z0-9\_\]+\[ \\t\]\*\\(\[ \\t\]\*(\[a-zA-Z0-9\_\]+)/\\1/d,definition/" --langmap=c++:+.c --exclude=testsuite -I GTY+ -I VEC+ -I DEF\_VEC\_P+ -R . ~/build/build-gcc-trunk/gcc
