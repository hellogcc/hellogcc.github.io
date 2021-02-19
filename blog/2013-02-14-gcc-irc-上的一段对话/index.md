---
title: "GCC IRC 上的一段对话"
date: "2013-02-14"
categories: 
  - "gcc"
---

今天无意中看到GCC IRC 上的一段对话，

\[21:34\] <richi> dnovillo: that said - introducing record-builder atthis point seems wrong \[21:34\] <dnovillo> what are the preconditions that you are thinking of? \[21:35\] <richi> that either everything "up" or everything "down" is C++-ified \[21:35\] <richi> C++-ifying random places makes GCC a mess \[21:35\] <dnovillo> i don't understand what that means. \[21:35\] <richi> datastructures are the very bottom \[21:35\] <richi> a GIMPLE pass would be the very top \[21:35\] <richi> but some pieces of tree? \[21:37\] <dnovillo> we are targeting specific facilities used by our teams.  that's our guiding principle.  they build new IL and types (asan).  our mandate does not include converting the whole codebase into C++, we don't have that kind of budget.

看来有些global maintainer对现在C++的修改也不满意。而且google目前并不打算把GCC的所有部分都用C++重写。希望我理解错了。
