---
title: "我向GCC社区提交过的Patch"
date: "2013-04-14"
categories: 
  - "gcc"
---

从2009年至今，23个Patch，共1512行。主要包括：

1、修复文档、注释、代码中的小错误 2、龙芯2F，3A等相关代码 3、其它遇到的问题

可以通过以下脚本生成上述所有的Patch。

`#! /bin/bash

############################################ # Generate patches submitted by me to GCC. # # Mingjie Xing (mingjie.xing@gmail.com) # ############################################

# r147644 - http://gcc.gnu.org/ml/gcc-cvs/2009-05/msg00620.html # r155645 - http://gcc.gnu.org/ml/gcc-cvs/2010-01/msg00104.html # r163028 - http://gcc.gnu.org/ml/gcc-cvs/2010-08/msg00239.html

# r163246 - http://gcc.gnu.org/ml/gcc-cvs/2010-08/msg00457.html # r163799 - http://gcc.gnu.org/ml/gcc-cvs/2010-09/msg00090.html # r163986 - http://gcc.gnu.org/ml/gcc-cvs/2010-09/msg00278.html # r166927 - http://gcc.gnu.org/ml/gcc-cvs/2010-11/msg00816.html # r167107 - http://gcc.gnu.org/ml/gcc-cvs/2010-11/msg00996.html # r167290 - http://gcc.gnu.org/ml/gcc-cvs/2010-11/msg01180.html # r167484 - http://gcc.gnu.org/ml/gcc-cvs/2010-12/msg00165.html # r168364 - http://gcc.gnu.org/ml/gcc-cvs/2010-12/msg01050.html # r168396 - http://gcc.gnu.org/ml/gcc-cvs/2011-01/msg00012.html # r168397 - http://gcc.gnu.org/ml/gcc-cvs/2011-01/msg00013.html # r168452 - http://gcc.gnu.org/ml/gcc-cvs/2011-01/msg00069.html # r170041 - http://gcc.gnu.org/ml/gcc-cvs/2011-02/msg00586.html # r170729 - http://gcc.gnu.org/ml/gcc-cvs/2011-03/msg00150.html # r170730 - http://gcc.gnu.org/ml/gcc-cvs/2011-03/msg00151.html # r171206 - http://gcc.gnu.org/ml/gcc-cvs/2011-03/msg00629.html # r174833 - http://gcc.gnu.org/ml/gcc-cvs/2011-06/msg00322.html # r181346 - http://gcc.gnu.org/ml/gcc-cvs/2011-11/msg00637.html # r187392 - http://gcc.gnu.org/ml/gcc-cvs/2012-05/msg00388.html # r187393 - http://gcc.gnu.org/ml/gcc-cvs/2012-05/msg00389.html # r190617 - http://gcc.gnu.org/ml/gcc-cvs/2012-08/msg00594.html

r=147644 && svn diff -c $r svn://gcc.gnu.org/svn/gcc > r$r.patch r=155645 && svn diff -c $r svn://gcc.gnu.org/svn/gcc > r$r.patch r=163028 && svn diff -c $r svn://gcc.gnu.org/svn/gcc > r$r.patch

r=163246 && svn diff -c $r svn://gcc.gnu.org/svn/gcc > r$r.patch r=163799 && svn diff -c $r svn://gcc.gnu.org/svn/gcc > r$r.patch r=163986 && svn diff -c $r svn://gcc.gnu.org/svn/gcc > r$r.patch r=166927 && svn diff -c $r svn://gcc.gnu.org/svn/gcc > r$r.patch r=167107 && svn diff -c $r svn://gcc.gnu.org/svn/gcc > r$r.patch r=167290 && svn diff -c $r svn://gcc.gnu.org/svn/gcc > r$r.patch r=167484 && svn diff -c $r svn://gcc.gnu.org/svn/gcc > r$r.patch r=168364 && svn diff -c $r svn://gcc.gnu.org/svn/gcc > r$r.patch r=168396 && svn diff -c $r svn://gcc.gnu.org/svn/gcc > r$r.patch r=168397 && svn diff -c $r svn://gcc.gnu.org/svn/gcc > r$r.patch r=168452 && svn diff -c $r svn://gcc.gnu.org/svn/gcc > r$r.patch r=170041 && svn diff -c $r svn://gcc.gnu.org/svn/gcc > r$r.patch r=170729 && svn diff -c $r svn://gcc.gnu.org/svn/gcc > r$r.patch r=170730 && svn diff -c $r svn://gcc.gnu.org/svn/gcc > r$r.patch r=171206 && svn diff -c $r svn://gcc.gnu.org/svn/gcc > r$r.patch r=174833 && svn diff -c $r svn://gcc.gnu.org/svn/gcc > r$r.patch r=181346 && svn diff -c $r svn://gcc.gnu.org/svn/gcc > r$r.patch r=187392 && svn diff -c $r svn://gcc.gnu.org/svn/gcc > r$r.patch r=187393 && svn diff -c $r svn://gcc.gnu.org/svn/gcc > r$r.patch r=190617 && svn diff -c $r svn://gcc.gnu.org/svn/gcc > r$r.patch`
