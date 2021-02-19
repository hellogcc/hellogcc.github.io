---
title: "邮件列表解读之STAMP-FILE"
date: "2012-01-16"
categories: 
  - "gcc"
---

来自gcc邮件列表，原文见[http://gcc.gnu.org/ml/gcc/2012-01/msg00090.html](http://gcc.gnu.org/ml/gcc/2012-01/msg00090.html)

GCC的Makefile里有许多类似的用法，形式如下：

FILE: STAMP-FILE; @true
STAMP-FILE: DEPENDENCIES
       commands to create FILE.tmp
       move-if-change FILE.tmp FILE
       touch $@

它的效果是，只有当FILE的内容发生变化时，依赖FILE的目标对象才会被重新构建。比如，如下片段：

\# all-tree.def includes all the tree.def files.
all-tree.def: s-alltree; @true
s-alltree: Makefile
        rm -f tmp-all-tree.def
        echo '#include "tree.def"' > tmp-all-tree.def
        echo 'END\_OF\_BASE\_TREE\_CODES' >> tmp-all-tree.def
        echo '#include "c-family/c-common.def"' >> tmp-all-tree.def
        ltf="$(lang\_tree\_files)"; for f in $$ltf; do \\
          echo "#include \\"$$f\\""; \\
        done | sed 's|$(srcdir)/||' >> tmp-all-tree.def
        $(SHELL) $(srcdir)/../move-if-change tmp-all-tree.def all-tree.def
        $(STAMP) s-alltree

1、首先，简单介绍下这种用法。

FILE: STAMP-FILE; @true

目标对象FILE依赖于STAMP-FILE。“; @true”是该规则要执行的命令。Makefile中第一条命令可以直接写在依赖后面。其相当于：

FILE: STAMP-FILE
        @true

true是shell命令，其作用是什么都不做，返回执行成功，即相当于空操作。“@”用于禁止显示命令输出。move-if-change是GCC源码包中自带的脚本，它会在进行mv操作之前，先进行对比，只有源文件和目标文件的内容不一样时，才执行mv。

所以，这段代码的意思是说如果DEPENDENCIES有更新，则会先创建FILE.tmp，如果FILE.tmp和FILE内容不一样则更新FILE，最后更新一下STAMP-FILE。

2、这里有两个疑惑。

首先，“; @true”去掉是否可以。答案是，对于GNU Make，是可以的。但是对于其它Make，未必可以。

其次，是否可以直接写成

FILE: DEPENDENCIES
       commands to create FILE.tmp
       move-if-change FILE.tmp FILE

这样，看起来也可以实现“只有当FILE的内容发生变化时，依赖FILE的目标对象才会被重新构建”的效果。

答案是，不可以的。问题在于如果DEPENDENCIES更新了，而FILE的内容没有变，则FILE的时间戳就不会更新，这样，每当运行make时，都会执行这条规则中的命令，执行不必要的操作。使用STAMP-FILE则只需要在DEPENDENCIES更新的时候，执行一次这条规则中的命令即可。
