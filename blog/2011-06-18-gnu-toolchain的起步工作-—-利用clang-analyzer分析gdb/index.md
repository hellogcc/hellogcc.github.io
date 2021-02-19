---
title: "GNU Toolchain的起步工作 — 利用clang-analyzer分析gdb"
date: "2011-06-18"
categories: 
  - "gdb"
---

GNU Toolchain的起步工作往往是最艰难的，因为往往有无从下手的感觉。开源社区往往有很多“潜规则”，使得一开始就进行一些大的修改，变得更加困难。所以，在刚刚开始GNU
Toolchain工作的时候，最好能够从一些小的地方入手，慢慢的融入这个社区（如果足够幸运，有global
maintainer领导你干一些事情，那些这个文章就没有用了，可以忽略）。本文以gdb为例，看看如何用一个静态检查工具，找到一些比较明显的程序错误。修改这些错误，就是一个很好的起点。

1 clang-analyzer
clang-analyzer是一个建立在llvm/clang基础上的一个静态检查工具。只要在需要检查的代码，例如gdb，用clang-analyzer来运行configure和make，就可以生成检查结果。我个人对clang-analyzer没有什么理解，如何编译参见[http://clang-analyzer.llvm.org/。](http://clang-analyzer.llvm.org/%E3%80%82)

2 检查gdb
clang-analyzer的优秀设计，使得不需要对gdb做任何调整，就可以对它进行静态分析。假如，我原有的gdb configure/make 如下，

$ ../git/gdb/configure --with-python=/usr/bin/python --disable-gdbtk
$ make -j4;

那么使用clang-analyzer检查gdb，就可以这样；

$ PATH=~/SourceCode/svn/build/Debug+Asser/bin:$PATH
~/SourceCode/svn/llvm/tools/clang/tools/scan-build/scan-build
../git/gdb/configure --with-python=/usr/bin/python --disable-gdbtk
$ PATH=~/SourceCode/svn/build/Debug+Asser/bin:$PATH
~/SourceCode/svn/llvm/tools/clang/tools/scan-build/scan-build make -j4

其实很简单，就是在原有的命令前边加入 scan-build.

原有的gdb build很变得比较慢，在gdb build结束后，你可以看到这样的输出

scan-build: 401 bugs found.
scan-build: Run 'scan-view /tmp/scan-build-2011-03-09-1' to examine bug reports.

你就运行scan-view就好了，
$ PATH=~/SourceCode/svn/build/Debug+Asserts/bin:$PATH
~/SourceCode/svn/llvm/tools/clang/tools/scan-view/scan-view
/tmp/scan-build-2011-03-09-1

3 分析gdb中的错误
在运行完scan-view后，浏览器会自动打开网页，里边包含clang-analyzer的分析结果，并且对找到的bug进行分类。首先声明一点，clang-analzyer和所有的静态分析一样，会有false
positive，所以，不是所有的错误都一定是真正的bug，需要我们自己结合程序逻辑自己分析。经过我自己的浏览，发现"Dead
Assignment"这样的错误基本上都是对的，因为这样的错误相对比较简单。

例如，
Dead store	Dead assignment	git /gdb /gdb /linux-nat.c	5603	1	View
Report 	Report Bug	Open File

单击 “View Report”，可以看到，pid在5603行写入以后，再没有在别的地方被读了。所以，这里\*有可能\*把5603行删去。

5596	struct address\_space \*
5597	linux\_nat\_thread\_address\_space (struct target\_ops \*t, ptid\_t ptid)
5598	{
5599	struct lwp\_info \*lwp;
5600	struct inferior \*inf;
5601	int pid;
5602
5603	pid = GET\_LWP (ptid)ptid\_get\_lwp (ptid);

Value stored to 'pid' is never read

5604	if (GET\_LWP (ptid)ptid\_get\_lwp (ptid) == 0)
5605	{
5606	/\* An (lwpid,0,0) ptid. Look up the lwp object to get at the
5607	tgid. \*/
5608	lwp = find\_lwp\_pid (ptid);
5609	pid = GET\_PID (lwp->ptid)ptid\_get\_pid (lwp->ptid);
5610	}
5611	else
5612	{
5613	/\* A (pid,lwpid,0) ptid. \*/
5614	pid = GET\_PID (ptid)ptid\_get\_pid (ptid);
5615	}
5616
5617	inf = find\_inferior\_pid (pid);
5618	gdb\_assert (inf != NULL)((void) ((inf != ((void\*)0)) ? 0 :
(internal\_error ("../../git/gdb/gdb/linux-nat.c"
, 5618, gettext ("%s: Assertion \`%s' failed."), \_\_PRETTY\_FUNCTION\_\_
, "inf != NULL"), 0)));
5619	return inf->aspace;
5620	}

这样，就可以自己写一个两三行的小patch，发送到gdb-patches了。如果运气好，你的第一个commit，就快有了 :)

4. 结束语
llvm有着十分优秀的infrastructure，使得在上边做一些静态分析工具，成为可能。clang-analyzer就是这样的一个小工具，而且在上边扩充十分容易。另外，我们可以用它来分析一些GNU
Toolchain，从中找到问题，提交我们自己的前几个patch。
