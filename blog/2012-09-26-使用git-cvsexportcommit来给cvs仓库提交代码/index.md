---
title: "使用git cvsexportcommit来给cvs仓库提交代码"
date: "2012-09-26"
categories: 
  - "git"
---

由于GDB还在使用古老的CVＳ，但是有一个git的镜像。平时工作的时候，就在git上，直到patch review完毕，需要commit的时候，才把patch apply到cvs trunk上，然后commit。这样每次都需要 _git format-patches_ 和 _patch -p1 < foo.patch_，很麻烦。

今天发现可以用 git cvsexportcommit 来把git repo里边的某个commit，apply到cvs本地。假如我的当前目录是GDB CVS checkout的source，我的GIT 目录是 ~/Source/gnu/gdb/git

\[yao@qiyao:~/Source/gnu/gdb/cvs/src\]$ export GIT\_DIR=~/Source/gnu/gdb/git/.git
_\[yao@qiyao:~/Source/gnu/gdb/cvs/src\]$ git cvsexportcommit -v 8121953ff81c6cdf36daca2ae797bbd124d01a91_ _Applying to CVS commit 8121953ff81c6cdf36daca2ae797bbd124d01a91 from parent 4a743c6b2ed69434f546a6559aca1819015ed00d_ _Checking if patch will apply_ _Enter passphrase for key '/home/yao/.ssh/id\_rsa':_ _Applying_ _Patch applied successfully. Adding new files and directories to CVS_ _Commit to CVS_ _Patch title (first comment line): Set breakpoint\_ops in mi\_cmd\_break\_insert._ _Ready for you to commit, just run:_ _cvs commit -F .msg 'gdb/breakpoint.h' 'gdb/mi/mi-cmd-break.c'_

这样以后，我就可以在添加了changelog以后，_cvs commit_了。注意在使用完了这个命令之后，最好把环境变量 **GIT\_DIR** 给 **unset**。不然在任何git repo下边，显示的都是这个目录里边的branches。
