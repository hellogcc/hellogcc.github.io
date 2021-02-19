---
title: "QEMU Internal – Block Chaining 3/3"
date: "2011-10-14"
categories: 
  - "qemu"
---

Copyright (c) 2011 陳韋任 (Chen Wei-Ren)

2\. Block Chaining

由 guest binary -> TCG IR 的過程中，gen\_goto\_tb 會做 block chaining 的準備。 我們先來看何時會呼叫到 gen\_goto\_tb。以 i386 為例，遇到 guest binary 中的條件分支和直接跳轉都會呼叫 gen\_goto\_tb (target-i386/translate.c)。這邊以條件分支當例子:

static inline void gen\_jcc(DisasContext \*s, int b,
                           target\_ulong val, target\_ulong next\_eip)
{
    int l1, l2, cc\_op;
 
    cc\_op \= s\->cc\_op;
    gen\_update\_cc\_op(s);
    if (s\->jmp\_opt) { // use direct block chaining for direct jumps
        l1 \= gen\_new\_label();
        gen\_jcc1(s, cc\_op, b, l1);
 
        gen\_goto\_tb(s, 0, next\_eip); // 我猜是 taken
 
        gen\_set\_label(l1);
        gen\_goto\_tb(s, 1, val); // 我猜是 not taken
        s\->is\_jmp \= DISAS\_TB\_JUMP;
    } else {
 
      /\* 忽略不提 \*/
 
    }
}

- gen\_goto\_tb。強烈建議閱讀 [Porting QEMU to Plan 9: QEMU Internals and Port Strategy](http://gsoc.cat-v.org/people/nwf/paper-strategy-plus.pdf) 2.2.3 和 2.2.4 節，也別忘了 [SOURCEARCHIVE.com](http://qemu.sourcearchive.com)。

// tb\_num 代表目前 tb block linking 分支情況。eip 代表跳轉目標。
static inline void gen\_goto\_tb(DisasContext \*s, int tb\_num, target\_ulong eip)
{
    TranslationBlock \*tb;
    target\_ulong pc;
 
    // s->pc 代表翻譯至目前 guest binary 的所在位址。tb->pc 表示 guest binary 的起始位址。
    // 注意! 這裡 s->cs\_base + eip 代表跳轉位址; s->pc 代表目前翻譯到的 guest pc。
    pc \= s\->cs\_base + eip; // 計算跳轉目標的 pc
    tb \= s\->tb; // 目前 tb
    // http://lists.nongnu.org/archive/html/qemu-devel/2011-08/msg02249.html
    // http://lists.gnu.org/archive/html/qemu-devel/2011-09/msg03065.html
    // 滿足底下兩個條件之一，則可以做 direct block linking
    // 第一，跳轉目標和目前 tb 起始的 pc 同屬一個 guest page。
    // 第二，跳轉目標和目前翻譯到的 pc 同屬一個 guest page。
    if ((pc & TARGET\_PAGE\_MASK) \== (tb\->pc & TARGET\_PAGE\_MASK) ||
        (pc & TARGET\_PAGE\_MASK) \== ((s\->pc \- 1) & TARGET\_PAGE\_MASK))  {
        // 如果 guest jump 指令和其跳轉位址同屬一個 guest page，則做 direct block linking。
 
        tcg\_gen\_goto\_tb(tb\_num); // 生成準備做 block linking 的 TCG IR。詳情請見之後描述。
 
        // 更新 env 的 eip，使其指向此 tb 之後欲執行指令的位址。
        // tb\_find\_fast 會用 eip 查找該 TB 是否已被翻譯過。
        gen\_jmp\_im(eip);
 
        // 最終回到 QEMU tcg\_qemu\_tb\_exec，賦值給 next\_tb。
        // 注意! tb\_num 會被 next\_tb & 3 取出，由此可以得知 block chaining 的方向。
        tcg\_gen\_exit\_tb((tcg\_target\_long)tb + tb\_num);
    } else {
        /\* jump to another page: currently not optimized \*/
        gen\_jmp\_im(eip);
        gen\_eob(s);
    }
}

- tcg\_gen\_goto\_tb 生成 TCG IR。

static inline void tcg\_gen\_goto\_tb(int idx)
{
    tcg\_gen\_op1i(INDEX\_op\_goto\_tb, idx);
}

- tcg\_out\_op (tcg/i386/tcg-target.c) 將 TCG IR 翻成 host binary。注意! 這邊利用 patch jmp 跳轉位址達成 block linking。

static inline void tcg\_out\_op(TCGContext \*s, TCGOpcode opc,
                              const TCGArg \*args, const int \*const\_args)
{
    case INDEX\_op\_goto\_tb:
        if (s\->tb\_jmp\_offset) {
            /\* direct jump method \*/
            tcg\_out8(s, OPC\_JMP\_long); /\* jmp im \*/
            // 紀錄將來要 patch 的地方。
            s\->tb\_jmp\_offset\[args\[0\]\] \= s\->code\_ptr \- s\->code\_buf;
            // jmp 的參數為 jmp 下一個指令與目標的偏移量。
            // 如果還沒做 block chaining，則 jmp 0 代表 fall through。
            tcg\_out32(s, 0);
        } else {
 
            /\* 在此忽略 \*/
 
        }
        // 留待將來 "reset" direct jump 之用。
        s\->tb\_next\_offset\[args\[0\]\] \= s\->code\_ptr \- s\->code\_buf;
        break;
}

回答上一篇最後留下的問題。在還未 patch code cache 中的分支跳轉指令的跳轉位址，它會 fall through，還記得 jmp 0 嗎? 我這邊在列出 gen\_goto\_tb 的部分內容:

tcg\_gen\_goto\_tb(tb\_num);
 
// Fall through
 
// 更新 env 的 eip，使其指向此 tb 之後欲執行指令的位址。
// tb\_find\_fast 會用 eip 查找該 TB 是否已被翻譯過。
gen\_jmp\_im(eip);
 
// 最終回到 QEMU tcg\_qemu\_tb\_exec，賦值給 next\_tb。
// 注意! tb\_num 會被 next\_tb & 3 取出，由此可以得知 block chaining 的方向。
tcg\_gen\_exit\_tb((tcg\_target\_long)tb + tb\_num);

目前執行的 tb 會賦值給 next\_tb (末兩位編碼 block chaining 的方向)。等待下一次迴圈，tb\_find\_fast 回傳 next\_tb 的下一個 tb。

if (next\_tb != 0 && tb\->page\_addr\[1\] \== \-1) {
    // 這邊利用 TranlationBlock 指針的最低有效位後兩位指引 block chaining 的方向。
    // next\_tb -> tb
    tb\_add\_jump((TranslationBlock \*)(next\_tb & ~3), next\_tb & 3, tb);
}

That’s it! That’s how direct block chaining is done in QEMU, I think… :-)
