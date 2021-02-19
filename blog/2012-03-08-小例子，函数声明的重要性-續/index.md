---
title: "小例子，函数声明的重要性 (續)"
date: "2012-03-08"
categories: 
  - "gcc"
---

Copyright (c) 2012 陳韋任 (Chen Wei-Ren)

人總是要經由教訓才能真正學到東西。我在[函数声明的重要性](http://www.hellogcc.org/archives/310)上深有體會。這裡給出我血淋淋的教訓。

情境描述:

在 x86\_64 的機器上，指針 (pointer) 大小為 64 bit。我自定一個函式如下，

  TranslationBlock \*my\_tb\_find\_pc(unsigned long tc\_ptr);

但是在使用上述函式的時候，我沒有先宣告其 prototype。

void tlb\_fill(target\_ulong addr, int is\_write, int mmu\_idx, void \*retaddr)
{
         ...
 
    TranslationBlock \*tb\_ptr \= my\_tb\_find\_pc(pc);
 
         ...
}

在用 GDB 進到 my\_tb\_find\_pc 觀察其返回值時，該值為 0x7fffe41b22d8。但回到 tlb\_fill 的時候，發現 tb\_ptr 被賦值 0xffffffffe41b22d8。這詭異的情況可以用以下流程解釋:

  0x7fffe41b22d8 -> (int32\_t ) 0xe41b22d8 -> (int64\_t) 0xffffffffe41b22d8

0x7fffe41b22d8 先被轉成 32 bit 的 int，再 sign extend 成 64 bit 的 int。這邊問題就在於根據 C90，編譯器會將沒有顯示宣告 prototype 的函式其返回值型別視為 int (32 bit)。這就會造成上述詭異的情況。開啟 -Wall 編譯會得到以下警告:

  warning: implicit declaration of function 'my\_tb\_find\_pc'
  warning: assignment makes pointer from integer without a cast

其訊息描述了編譯器將沒有顯式宣告 prototype 的 my\_tb\_find\_pc 其返回值型別視為 int，再將其轉為指針 (pointer)。
