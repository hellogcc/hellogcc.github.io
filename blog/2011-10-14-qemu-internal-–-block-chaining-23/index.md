---
title: "QEMU Internal – Block Chaining 2/3"
date: "2011-10-14"
categories: 
  - "qemu"
---

Copyright (c) 2011 陳韋任 (Chen Wei-Ren)

2\. Block Chaining

我們再回到 cpu\_exec (cpu-exec.c) 的內層迴圈。這邊主要看 tb\_add\_jump。

if (next\_tb != 0 && tb\->page\_addr\[1\] \== \-1) {
    // 這邊利用 TranlationBlock 指針的最低有效位後兩位指引 block chaining 的方向。
    // next\_tb -> tb
    tb\_add\_jump((TranslationBlock \*)(next\_tb & ~3), next\_tb & 3, tb);
}
 
// 這邊要注意到，QEMU 利用 TranslatonBlock 指針後兩位必為零的結果
// 做了一些手腳。QEMU 將其末兩位編碼成 0、1 或 2 來指引 block chaing
// 的方向。這種技巧在 QEMU 用得非常嫻熟。 
next\_tb \= tcg\_qemu\_tb\_exec(tc\_ptr);

- tb\_add\_jump 呼叫 tb\_set\_jmp\_target 做 block chaining 的 patch。另外，會利用 tb 的 jmp\_next 和 tb\_next 的 jmp\_first 把 block chaining 中的 TranslationBlock 串成一個 circular list。這是以後做 block \*unchaining\* 之用。再複習一下 TranslationBlock，
    
        struct TranslationBlock\*        code cache (host binary)
                tb->tc\_ptr        ->            TB
    
    請注意! 當我們講 TB，可能是指 TranslationBlock，也可能是指 code cache 中的 TB。好! 當我們 patch code cache 中的 TB，將它們串接在一起。我們同時也要把它們相對應的 TranslationBlock 做點紀錄，利用 jmp\_first 和 jmp\_next 這兩個欄位。
    
    這邊我先回到 tb\_link\_page，還記得我留下一些地方沒講嘛? 開始吧! ;-)
    
    // jmp\_first 代表跳至此 tb 的其它 TB 中的頭一個。jmp\_first 初值為自己，末兩位為 10 (2)。
    // 將來做 block chaining 時，jmp\_first 指向跳至此 tb 的其它 TB 中的頭一個，tb1，末兩位
    // 為 00 或 01，這代表從 tb1 的哪一個分支跳至此 tb。
    tb\->jmp\_first \= (TranslationBlock \*)((long)tb | 2);
     
    // jmp\_next\[n\] 代表此 tb 條件分支的目標 TB。
    // 注意! 如果目標 TB，tb2，孤身一人，jmp\_next 就真的指向 tb2 (符合初次看到 jmp\_next 所想
    // 的語意)。
    //
    // 如果已有其它 TB，tb3，跳至 tb2，則賦值給 tb->jmp\_next 的是 tb2 的 jmp\_first，也就是
    // tb3 (末兩位編碼 tb3 跳至 tb2 的方向)。
    tb\->jmp\_next\[0\] \= NULL;
    tb\->jmp\_next\[1\] \= NULL;
     
    // tb\_next\_offset 代表此 TB 在 code cache 中分支跳轉要被 patch 的位址 (相對於其 code cache
    // 的偏移量)，為了 direct block chaining 之用。
    if (tb\->tb\_next\_offset\[0\] != 0xffff)
        tb\_reset\_jump(tb, 0);
    if (tb\->tb\_next\_offset\[1\] != 0xffff)
        tb\_reset\_jump(tb, 1);
    
    我們再回來看看 tb\_add\_jump 做了什麼。:-)
    
    // block chaining 方向為: tb -> tb\_next。n 用來指示 tb 條件分支的方向。
    static inline void tb\_add\_jump(TranslationBlock \*tb, int n,
                                   TranslationBlock \*tb\_next)
    {
        // jmp\_next\[0\]/jmp\_next\[1\] 代表 tb 條件分支的目標。
        if (!tb\->jmp\_next\[n\]) {
            /\* patch the native jump address \*/
            tb\_set\_jmp\_target(tb, n, (unsigned long)tb\_next\->tc\_ptr);
     
            // tb\_jmp\_remove 會用到 jmp\_next 做 block unchaining，這個以後再講。
     
            // tb\_next->jmp\_first 初值為自己，末兩位設為 10 (2)。
            // 如果已有其它 TB，tb1，跳至 tb\_next，則下面這條語句會使得
            // tb->jmp\_next 指向 tb1 (末兩位代表 tb1 跳至 tb\_next 的方向)。
            // tb\_next->jmp\_first 由原本指向 tb1 改指向 tb。
            // 有沒有 circular list 浮現在你腦海中? ;-)
            tb\->jmp\_next\[n\] \= tb\_next\->jmp\_first;
     
            // tb\_next 的 jmp\_first 指回 tb，末兩位代表由 tb 哪一個條件分支跳至 tb\_next。
            tb\_next\->jmp\_first \= (TranslationBlock \*)((long)(tb) | (n));
        }
    }
    

我們再來看是怎麼 patch code cache 中分支指令的目標地址。依據是否採用 direct jump，tb\_set\_jmp\_target (exec-all.h) 有不同做法。採用 direct jump 的話，tb\_set\_jmp\_target 會根據 host 呼叫不同的 tb\_set\_jmp\_target1。tb\_set\_jmp\_target1 會用到 TB 的 tb\_jmp\_offset。如果不採用 direct jump 做 block chaining，tb\_set\_jmp\_target 會直接修改 TB 的 tb\_next。

- tb\_set\_jmp\_target (exec-all.h)。

static inline void tb\_set\_jmp\_target(TranslationBlock \*tb,
                                     int n, unsigned long addr)
{
    unsigned long offset;
 
    // n 可以為 0 或 1，代表分支跳轉的分向 taken 或 not taken。 
    offset \= tb\->tb\_jmp\_offset\[n\]; // tb 要 patch 的位址相對於 tb->tc\_ptr 的偏移量。
    tb\_set\_jmp\_target1((unsigned long)(tb\->tc\_ptr + offset), addr);
}

- tb\_set\_jmp\_target1 (exec-all.h)。

#elif defined(\_\_i386\_\_) || defined(\_\_x86\_64\_\_)
static inline void tb\_set\_jmp\_target1(unsigned long jmp\_addr, unsigned long addr)
{
    /\* patch the branch destination \*/
    // jmp 的參數為 jmp 下一條指令與目標地址的偏移量。
    \*(uint32\_t \*)jmp\_addr \= addr \- (jmp\_addr + 4);
    /\* no need to flush icache explicitly \*/
}

你會有這個疑問嗎? 在分支跳轉位址被 patch 到分支跳轉指令之前，它是要跳去哪裡? :-)

[QEMU – block chaining](http://www.hellogcc.org/wp-content/uploads/2011/10/QEMU-block-chaining.ppt) 可以幫助你比較清楚了解 block chaining 如何運作。
