---
title: "Statistics of GDB Commits"
date: "2011-11-03"
categories: 
  - "gdb"
  - "杂谈"
---

今天下午无意中想看看自己有了多少个commit，然后就用 _git log_ 命令查看，发现速度不错。不知道怎么了，灵机一动，想想看看这些global maintainer有多少commit。就有了如下的pie chart。  

我来解释一下这个图是怎么得到的。首先我

_git log gdb/ | grep ^commit | wc -l_ 统计一共gdb有多少次commit。然后把每个global maintainer的名字输入到这样的命令  
_git log gdb/ –author=”NAME” | grep ^commit | wc -l_

统计每个global maintainer的commit数量。然后把这些数据画出来，这个时候想起了google chart api，以前同事用过，觉得很fancy，我就自己copy了一个。

从这个图表里边，能看出很多有意思的事情，

gdbadmin 有很多commit。这些commit大多来自 daily version bump up，其实没有任何实际意义的。

来自other的部分占约1/4。这部分人可以认为是对gdb代码不是那么了解的人。

commit最多的是Andrew Cagney。 此人很牛，以前是Redhat GDB的头，后来去做frysk，然后离开了RedHat。 他那个时候，GDB好像还没有现在的 SC 和 Global Maintainer 机制。就他一个，叫 Head Developer。所以，他的commit最多。

commit 其次多的是 Mark K.，Daniel J.，Joel B. Michael S.。 Michael 已经去世了，Mark K. 很久很久没有commit了。 Daniel J. 貌似 08 年以后也没有什么commit了。 Joel B. 现在的 Release Manager，最近没有什么patch，最近的两个patch，我觉得也不怎么样。貌似是没有时间。

现在活跃的global maintainer 都有 400到 800个commit，所以，这个能告诉我们什么呢？ 就是我们把这句话反着读，就是，如果有了 400到 800个commit， 基本上就能当global maintainer。
