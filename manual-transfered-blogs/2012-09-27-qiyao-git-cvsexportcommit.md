转换时间： 2021-01-31

# 使用git cvsexportcommit来给cvs仓库提交代码
Posted on 2012/09/27 by yao

由于GDB还在使用古老的CVS，但是有一个git的镜像。平时工作的时候，就在git上，直到patch review完毕，需要commit的时候，才把patch apply到cvs trunk上，然后commit。这样每次都需要 git format-patches 和 patch -p1 < foo.patch，很麻烦。

今天发现可以用 git cvsexportcommit 来把git repo里边的某个commit，apply到cvs本地。假如我的当前目录是GDB CVS checkout的source，我的GIT 目录是 ~/Source/gnu/gdb/git
```
[yao@qiyao:~/Source/gnu/gdb/cvs/src]$ export GIT_DIR=~/Source/gnu/gdb/git/.git
[yao@qiyao:~/Source/gnu/gdb/cvs/src]$ git cvsexportcommit -v 8121953ff81c6cdf36daca2ae797bbd124d01a91
Applying to CVS commit 8121953ff81c6cdf36daca2ae797bbd124d01a91 from parent 4a743c6b2ed69434f546a6559aca1819015ed00d
Checking if patch will apply
Enter passphrase for key '/home/yao/.ssh/id_rsa':
Applying
Patch applied successfully. Adding new files and directories to CVS
Commit to CVS
Patch title (first comment line): Set breakpoint_ops in mi_cmd_break_insert.
Ready for you to commit, just run:
cvs commit -F .msg 'gdb/breakpoint.h' 'gdb/mi/mi-cmd-break.c'
```
这样以后，我就可以在添加了changelog以后，cvs commit了。注意在使用完了这个命令之后，最好把环境变量 GIT_DIR 给 unset。不然在任何git repo下边，显示的都是这个目录里边的branches。
