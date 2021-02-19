---
title: "开始加入gnu toolchain的开发"
date: "2011-06-18"
categories: 
  - "杂谈"
---

作者： qiyao@hellogcc

很多刚刚加入gnu toolchain开发的工程师，都会被各种各样的规定，缩写还有交流方式搞得晕头转向。本文就是为了让刚刚进入gnu toolchain的开发的工程师简单介绍一下在这个圈子里边工作的一些特别之处。

  
1 开发流程  
checkout change diff submit approve commit

checkout就不用说了，绝大多数人都知道怎样check out代码。

change:  
这里以emacs为例，编写代码要注意gnu的编码规范。写代码前，自己好好读两遍。

对于其他编辑器，比如vim和eclipse，用户可以自行设置。只要符合下边的一些要求就可以了

1 敲击一下tab是两个空格，  
2 如果前边有了八个空格，用一个tab替代，  
3 一般源程序的宽度不要超过76或者78，  
4 changelog的宽度不要超过72，

这里有一个比较详细的，[http://sourceware.org/gdb/wiki/JoelsCodingStyleCheatSheet](http://sourceware.org/gdb/wiki/JoelsCodingStyleCheatSheet)  
  
虽然是给gdb开发者看的，但是基本适合其他gnu toolchain项目。

建议安装whitespace.el，来显示空格和tab，还可以去掉一些trail space。还有在使用git生成patch的时候，使用git diff –check，可以帮助你检查patch。

生成patch以后，一般是在patch的头部加入changelog，用[mklog.pl](http://mklog.pl/)  
。这个时候不用修改源代码的changlog。

发送patch，一般gnu toolchain都用单独的邮件列表来讨论patch的。所以，把你的patch按照附件的形式，用纯文本的方式，放送到列表。邮件的题目要简单明了，因为邮件列表的流量很大，而且每个maintainer可能只注意自己需要review的patch，所以标题要让他/她知道，这个是他/她需要review的。比如，有一个patch是对arm的修改，可以这样\[patch/arm\] XXXXX。多看看别人怎样写的。

  
还有的时候，邮件标题可以用 \[rfc\], \[rfa\]之类的。rfc的意思是request for commit，而rfa的意思是request for  
approval/accept。我自己并不经常用这两个，我个人感觉他们的差别在于rfc是提交自己的一个代码或者patch，希望被接受；而rfa是提交的自己的想法或者建议。比如我想建议在gdb中做一个新的feature，program slicing，我可以把我的建议和设计这样发出去 \[rfa\] New feature in GDB : program slicing

\[note: 我个人对rfc和rfa用的不多，所以解释不是很准确\]

在邮件中简单介绍你的修改的目的和概要，如果需要，保证这个修改没有引入regression。

有的时候，可能为了实现一个功能，修改是分很多部分的，比如第一部分是为了增加这样的功能，对现有代码的重构，第二部分是实现新功能的代码，第三部分是测试用例，第四部分是文档。如果作为一个patch发出去，别人review起来比较难受，也很难理解你的意图，这个时候，就要在一个邮件thread里边，把patch分多个发出去。具体操作如下，在发patch之前，自己先想好，如何把自己的修改合理的拆分到若干个patch里边去，然后第一封信写清楚你这一系列patch的目的，和大体实现，不要attach任何patch，比如  
\[patch 0\] GDB new feature: program slicing，发送。然后依次恢复这封邮件，分别按照不同的题目，把自己的patch按照顺序发出去，比如  
\[patch 1\] refactor existing GDB infrun to prepare for program slicing  
\[patch 2\] support program slicing in target-independent part  
\[patch 3/x86\] support program slicing on x86  
\[patch 4\] test case and doc

这样的好处就是，负责doc和testcase的maintainer就需要看你的第三个patch就好了。负责x86 maintainer就看你的patch 2。而且以后给comments也比较容易。

如果还不明白，看看别人是怎样写patch的。不过还不明白，可以在hellogcc讨论。

commit的时候，加入changelog entry。在emacs里边，比较方便，M-x add-changelog-entry后，选择changlog文件，就会自动加入你的邮件地址和时期，完全符合gnu的规定。如果源代码不是很多，比如gdb，可以用emacs提供的前端pcl-cvs, pcl-git来替代命令行的操作。 M-x cvs-examine: 选择本地的cvs checkout出来的代码，等待一会后，emacs中会有一个cvs summary的buffer，显示本地目录那些改动过的文件。在提交的时候，用m选中那些需要提交的文件，然后按c，就会出现一个新buffer，里边是填写commit log。写完commit log后 C-c C-c,就完成commit了。pcl-cvs和pcl-git还有很多别的用法，可以参考网上的文章。

测试:  
不要忽略自动测试的部分。个人感觉，gdb里边写的做好的部分就是testsuite了 :)  
gcc和gdb的wiki上都有一页关于编写测试用例的。建议好好读一读。如果之前对dejagnu和tcl/expect不是很清楚的人，建议去了解一下dejagnu。

regression test:如果你持续在某个toolchain上开发，最好有一个nightly build/test的环境，这里就不介绍持续测试和持续集成了。因为gnu toolchain每天都有来自不同组织处于不同目的对软件进行修改，所以，难免影响到你所希望的功能。如果发现了有regression，可以自己试试修改，这也是做贡献的一种方式。

2 参与开发  
gnu toolchain都比较庞大，支持的平台很多，所以刚开始理解代码并且加入到开发是一个十分困难的事情，但是这个也是没有其它什么捷径。造成这些困难的有如下一些因素

历史悠久: 历史悠久对于一个国家或者城市来说，是一个好事情，但是，对于软件来说，这绝对不是什么好事情。

支持很多平台:

缺少文档:

开放的是代码而不是设计: 我们可以看到代码，但是很难看到设计。设计往往存在于代码的注释或者当时邮件列表里边的讨论。

寻找帮助:  
由于开源软件都缺乏文档，所以碰到不会的问题，就要在社区中寻求帮助。目前，社区的交流方式有这样几种，邮件和irc。当然还有第三种，就是参加一年一次的gcc summit，但是这个对大多数人是不适用的。

邮件列表:是开发人员的主要交流手段。如何写邮件可能是另外一个话题了，我这里简单介绍一下。其实和考研写英语作文很像，就是开始就要明确地告诉别人，我需要什么样的帮助，要非常具体。这样被人才能帮助你。我们这样想，我们都希望得到大牛的指点，但是，大牛一天要收多少邮件呢？凡是他/她第一感觉能回的邮件，他就马上回了，如果有点模糊，他/她可能就跳过去了，回复的可能性很小。所以开门见山，告诉别人你的问题是什么，你都做了那些尝试，还有那些不明白，需要别人的指点。

irc:irc是一种即时聊天软件。我个人觉得，不要期望你能在irc上得到十分好的帮助，除非你和irc的人都很熟了。因为，gnu toolchain的问题，都不是一句两句能说清楚的事情(除非你问gcc 4.6.0什么发布，irc也是一个不错的地方)，所以我个人认为irc在获得帮助上没有太大的用处。有一种情况是irc有用的，就是你和某一位或者几位开发人员在邮件列表上已经有过一些交流，剩下一些细节或者yes/no的问题，在irc上问一下，也还不错。

贡献:  
其实在open source 社区中，贡献这个词语出现很多次，但是我们还是没有很多的贡献，至少我是这样。下边这些是为那些想为open source 做些事情的人写的，如果你只是为了用开源软件完成自己的工作或者研究项目，可以跳过这一段。首先澄清一个概念，不是只有global maintainer才可以做贡献，我们这些刚刚开始做的人，也可以。勿以贡献小而不为。我能够想到的，在社区的贡献对你有以下的帮助  
1 慢慢被社区所认可和接受，这样对于以后的工作会更有帮助，也有助于下一步的发展  
2 短期来讲，有助于别人帮助review你的patch  
3 提高在社区的reputation

对于我们刚刚起步的人，可以做那些贡献呢？我们可以到gdb的wiki上看看，上边说了一些需要帮忙的领域，在gdb邮件列表archive里边找找相关的讨论，就能找到一些事情。这样的话，你将来碰到问题，也有人愿意帮助你。所以，尽力去发现开源软件的不足，尽力去改进它，虽然我们很多人在gnu toolchain上开发是我们的工作，但是不要把自己的贡献局限于我们的工作。

总结起来就是，眼光要放远，但是从细节着手。不要把自己的眼光之放在自己的工作的领域，要看得更宽一些，如果有机会在别的部分做一些东西，这样也有助于你有一个大局观。从细节着手，就是说，gnu toolchain的代码都很多，刚开始根本找不到要该的地方。这个时候，就要从一些小的地方开始，比如修一些bug，慢慢的，就会知道更多要改进的地方了。

总结:  
罗马不是一天建成的。我们也不可能在很短的时间内熟悉这个community，更不要奢望读了这篇blog就可以熟悉这个community的工作方式。我想，只要我们保持热情去贡献，增加交流，在我们中国诞生global maintainer也不是不可能的事情。
