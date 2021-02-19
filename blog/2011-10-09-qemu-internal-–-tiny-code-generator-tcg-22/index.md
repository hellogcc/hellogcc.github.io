---
title: "QEMU Internal – Tiny Code Generator (TCG) 2/2"
date: "2011-10-09"
categories: 
  - "qemu"
---

Copyright (c) 2011 陳韋任 (Chen Wei-Ren)

1.2 TCG Flow

介紹完一些資料結構之後，我開始介紹 cpu\_exec 的流程。底下複習一下 process mode 的流程:

cpu\_loop (linux-user/main.c) -> cpu\_x86\_exec/cpu\_exec (cpu-exec.c)

cpu\_exec 有兩層 for 迴圈。我們先看內層:

next\_tb \= 0; /\* force lookup of first TB \*/
for(;;) {
 
    tb \= tb\_find\_fast();
 
    tc\_ptr \= tb\->tc\_ptr;
 
    next\_tb \= tcg\_qemu\_tb\_exec(tc\_ptr);
}

- tb\_find\_fast 會先試圖查看目前 pc (guest virtual address) 是否已有翻譯過的 host binary 存放在 code cache。

// pc = eip + cs\_base
cpu\_get\_tb\_cpu\_state(env, &pc, &cs\_base, &flags);
 
// CPUState 中的 tb\_jmp\_cache 即是做此用途。
tb \= env\->tb\_jmp\_cache\[tb\_jmp\_cache\_hash\_func(pc)\];
 
// 檢查該 tb 是否合法。這是因為不同的 eip + cs\_base 可能會得到相同的 pc。
if (unlikely(!tb || tb\->pc != pc || tb\->cs\_base != cs\_base ||
               tb\->flags != flags)) {
        tb \= tb\_find\_slow(pc, cs\_base, flags);
}
 
// code cache 已有翻譯過的 host binary，返回 TranslationBlock。
return tb;

- tb\_find\_slow 以 pc 對映的物理位址 (guest physcal address) 查找 TB。如果成功，則將該 TB 寫入 env->tb\_jmp\_cache; 若否，則進行翻譯。

phys\_pc \= get\_page\_addr\_code(env, pc);
phys\_page1 \= phys\_pc & TARGET\_PAGE\_MASK;
 
// 除了 env->tb\_jmp\_cache 這個以 guest virtual address 為索引的緩存之外，
// QEMU 還維護了一個 tb\_phys\_hash，這個是以 guest physical address 為索引。
h \= tb\_phys\_hash\_func(phys\_pc);
ptb1 \= &tb\_phys\_hash\[h\];
for (;;) {
 
  not\_found:
 
  found:
 
  // TranslationBlock 中的 phys\_hash\_next 用在這裡。
  // 如果 phys\_pc 索引到同一個 tb\_phys\_hash 欄位，用 phys\_hash\_next 串接起來。
  ptb1 \= &tb\->phys\_hash\_next;
}
 
not\_found:
  tb \= tb\_gen\_code(env, pc, cs\_base, flags, 0);
 
found:
  env\->tb\_jmp\_cache\[tb\_jmp\_cache\_hash\_func(pc)\] \= tb;
  return tb;

這裡小結一下流程。

cpu\_exec (cpu-exec.c) -> tb\_find\_fast (cpu-exec.c)
  -> tb\_find\_slow (cpu-exec.c)

QEMU 先以 guest virtual address (GVA) 查找是否已有翻譯過的 TB，再以 guest physical address (GPA) 查找是否已有翻譯過的 TB。

如果沒有翻譯過的 TB，開始進行 guest binary -> TCG IR -> host binary 的翻譯。大致流程如下:

tb\_gen\_code (exec.c) -> cpu\_gen\_code (translate-all.c)
  -> gen\_intermediate\_code (target-i386/translate.c)
  -> tcg\_gen\_code (tcg/tcg.c) -> tcg\_gen\_code\_common (tcg/tcg.c)

- tb\_gen\_code 配置內存給 TB (TranslationBlock)，再交由 cpu\_gen\_code。

// 注意! 這裡會將 GVA 轉成 GPA。phys\_pc 將交給之後的 tb\_link\_page 使用。
phys\_pc \= get\_page\_addr\_code(env, pc);
tb \= tb\_alloc(pc);
if (!tb) {
  // 清空 code cache
}
 
// 初始 tb
 
// 開始 guest binary -> TCG IR -> host binary 的翻譯。
cpu\_gen\_code(env, tb, &code\_gen\_size);
 
// 將 tb 加入 tb\_phys\_hash 和二級頁表 l1\_map。
// phys\_pc 和 phys\_page2 分別代表 tb (guest pc) 對映的 GPA 和所屬的第二個
// 頁面 (如果 tb 代表的 guest binary 跨頁面的話)。
tb\_link\_page(tb, phys\_pc, phys\_page2);
return tb;

我底下分別針對 cpu\_gen\_code 和 tb\_link\_page 稍微深入的介紹一下。

- cpu\_gen\_code 負責 guest binary -> TCG IR -> host binary 的翻譯。

// 初始 TCGContext 的 gen\_opc\_ptr 和 gen\_opparam\_ptr，使其分別指向
// gen\_opc\_buf 和 gen\_opparam\_buf。gen\_opc\_buf 和 gen\_opparam\_buf
// 分別存放 TCGOpcode 和 operand。
tcg\_func\_start(s);
 
// 呼叫 gen\_intermediate\_code\_internal 產生 TCG IR
gen\_intermediate\_code(env, tb);
 
// TCG IR -> host binary
gen\_code\_size \= tcg\_gen\_code(s, gen\_code\_buf);

- gen\_intermediate\_code\_internal (target-\*/translate.c) 初始化並呼叫 disas\_insn 反組譯 guest binary 成 TCG IR。disas\_insn 呼叫 tcg\_gen\_xxx (tcg/tcg-op.h) 產生 TCG IR。分別將 opcode 寫入 gen\_opc\_ptr 指向的緩衝區 (translate-all.c 裡的 gen\_opc\_buf); operand 寫入 gen\_opparam\_ptr 指向的緩衝區 (translate-all.c 裡的 gen\_opparam\_buf)。
- tcg\_gen\_code (tcg/tcg.c) 呼叫 tcg\_gen\_code\_common (tcg/tcg.c) 將 TCG IR 轉成 host binary。

tcg\_reg\_alloc\_start(s);
 
s\->code\_buf \= gen\_code\_buf;
// host binary 會寫入 TCGContext s 的 code\_ptr 所指向的緩衝區。
s\->code\_ptr \= gen\_code\_buf;

至此，guest binary -> TCG IR -> host binary 算是完成了。剩下把 TranslationBlock (TB) 納入 QEMU 的管理，這是 tb\_link\_page 做的事。

- tb\_link\_page (exec.c) 把新的 TB 加進 tb\_phys\_hash 和 l1\_map 二級頁表。 tb\_find\_slow 會用 pc 對映的 GPA 的哈希值索引 tb\_phys\_hash。

/\* 把新的 TB 加進 tb\_phys\_hash \*/
h \= tb\_phys\_hash\_func(phys\_pc);
ptb \= &tb\_phys\_hash\[h\];
// 如果兩個以上的 TB 其 phys\_pc 的哈希值相同，則做 chaining。
tb\->phys\_hash\_next \= \*ptb;
\*ptb \= tb; // 新加入的 TB 放至 chaining 的開頭。
 
// 在 l1\_map 中配置 PageDesc 給 TB，並設置 TB 的 page\_addr 和 page\_next。
tb\_alloc\_page(tb, 0, phys\_pc & TARGET\_PAGE\_MASK);
if (phys\_page2 != \-1) // TB 對應的 guest binary 跨頁
    tb\_alloc\_page(tb, 1, phys\_page2);
else
    tb\->page\_addr\[1\] \= \-1;
 
// 以下和 block chaining 有關，留待下次再講，這邊暫且不提。
tb\->jmp\_first \= (TranslationBlock \*)((long)tb | 2);

- tb\_alloc\_page (exec.c) 設置 TB 的 page\_addr 和 page\_next，並在 l1\_map 中配置 PageDesc 給 TB。

static inline void tb\_alloc\_page(TranslationBlock \*tb,
                                 unsigned int n, tb\_page\_addr\_t page\_addr)
{
  // 代表 tb (guest binary) 所屬 guest page。
  tb\->page\_addr\[n\] \= page\_addr;
  // 在 l1\_map 中配置一個 PageDesc，返回該 PageDesc。
  p \= page\_find\_alloc(page\_addr \>> TARGET\_PAGE\_BITS, 1);
  // 將該頁面目前第一個 TB 串接到此 TB。將來有需要將某頁面所屬所有 TB 清空。
  tb\->page\_next\[n\] \= p\->first\_tb;
  // n 為 1 代表 tb 對應的 guest binary 跨 page。
  p\->first\_tb \= (TranslationBlock \*)((long)tb | n);
  // PageDesc 會維護一個 bitmap，這是給 SMC 之用。這裡不提。
  invalidate\_page\_bitmap(p);
}

這裡先回顧一下，QEMU 查看當前 env->pc 是否已翻譯過。若否，則進行翻譯。

tb\_find\_fast (cpu-exec.c) -> tb\_find\_slow (cpu-exec.c)
  -> tb\_gen\_code (exec.c)

tb\_gen\_code 講到這裡，guest binary -> host binary 已翻譯完成，相關資料結構也已設置完畢。返回 TB (TranslationBlock \*) 給 tb\_find\_fast。

tb \= tb\_find\_fast();
 
tc\_ptr \= tb\->tc\_ptr; // tc\_ptr 指向 code cache (host binary)
 
next\_tb \= tcg\_qemu\_tb\_exec(tc\_ptr);

很好，我們準備從 QEMU 跳入 code cache 開始執行了。:-) tcg\_qemu\_tb\_exec 被定義在 tcg/tcg.h。

#define tcg\_qemu\_tb\_exec(tb\_ptr) ((long REGPARM (\*)(void \*))code\_gen\_prologue)(tb\_ptr)

`(long REGPARM (*)(void *))` 將 code\_gen\_prologue 轉型成函式指針，`void *` 為該函式的參數，返回值為 long。REGPARM 指示 GCC 此函式透過暫存器而非棧傳遞參數。至此，`(long REGPARM (*)(void *))` 將數組指針 code\_gen\_prologue 轉型成函式指針。tb\_ptr 為該函式指針的參數。綜合以上所述，code\_gen\_prologue 被視為一函式，其參數為 tb\_ptr，返回當前 TB (tc\_ptr 代表的 TB，等講到 block chaining 會比較清楚)。code\_gen\_prologue 所做的事為一般函式呼叫前的 prologue，之後將控制交由 tc\_ptr 指向的 host binary 並開始執行。
