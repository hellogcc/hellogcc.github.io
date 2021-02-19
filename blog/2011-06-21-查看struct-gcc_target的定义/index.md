---
title: "查看struct gcc_target的定义"
date: "2011-06-21"
categories: 
  - "gcc"
---

xmj@hellogcc 现在结构体struct gcc\_target的定义，是通过一系列的宏来实现了，相关文件包括：target.h，target.def，target-hooks-def.h。这些宏，包括 #define DEFHOOKPOD(NAME, DOC, TYPE, INIT) TYPE NAME; #define DEFHOOK(NAME, DOC, TYPE, PARAMS, INIT) TYPE (\* NAME) PARAMS; #define HOOK\_VECTOR\_1(NAME, FRAGMENT) HOOKSTRUCT(FRAGMENT) #define HOOK\_VECTOR(INIT\_NAME, SNAME) HOOK\_VECTOR\_1 (INIT\_NAME, struct SNAME {) #define HOOK\_VECTOR\_END(DECL\_NAME) HOOK\_VECTOR\_1(,} DECL\_NAME ; ) 。。。 如果，要实现，增加一个target hook，就需要在target.def文件里增加相应的条目。为了能够查看实际的结构体定义，可以在trunk/gcc目录下，运行命令 $ gcc -E target.h -I ../../build-trunk/gcc/ -o ~/temp/target.i 查看~/temp/target.i文件，可以看到struct gcc\_target的定义： struct gcc\_target { struct [\[...\]](http://www.hellogcc.org/archives/109)
