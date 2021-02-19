---
title: "RFA 和 RFC的缩写"
date: "2012-10-14"
categories: 
  - "gdb"
  - "社区活动"
tags: 
  - "gdb-2"
---

我们平时在mail list里边经常看到 rfa 和 rfc，不清楚它们的含义。我之前也只是大概知道，具体的意思也不很清楚，今天无意中看到了一个[mail](http://sources.redhat.com/ml/gdb-patches/2003-01/msg00149.html "mail")，里边有对这些的解释：

- RFC == "request for comments". The submitter wants feedback on the patch and may not even want to commit it yet.
- RFA == "request for approval". The submitter has little doubt that the patch is ready to commit, and just wants a short answer.
- PATCH == "it's going in". The submitter has authority to commit this patch and is about to do so, either immediately or as soon as they get around to it. No reply is needed.

RFC -> RFA -> PATCH 就是，作者对自己的patch的信心越来越高。比如做了一个新feature，但是想听听社区对这个东西认可程度，可以用rfc。如果自己相信这样的patch就是正确的fix，就可以用fix。这里对PATCH的解释有些和现在理解的不太一样；我都现在的用法理解是，rfa 和 patch基本差不多。
