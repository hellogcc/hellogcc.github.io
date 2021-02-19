---
title: "QEMU Internal – Block Chaining 1/3"
date: "2011-10-14"
categories: 
  - "qemu"
---

Copyright (c) 2011 陳韋任 (Chen Wei-Ren)

2\. Block Chaining

先複習一下 QEMU 的流程，目前的情況如底下這樣:

    QEMU -> code cache -> QEMU -> code cache -> ...

之前提到，QEMU 是以一個 translation block 為單位進行翻譯並執行。這也就是說，每當在 code cache 執行完一個 translation block 之後，控制權必須交還給 QEMU。這很沒有效率。理想情況下，大部分時間應該花在 code cache 中執行，只要在必要情況下才需要回到 QEMU。基本想法是把 code cache 中的 translation block 串接起來。只要 translation block 執行完之後，它的跳躍目標確定且該跳躍目標也已經在 code cache 裡，那我們就把這兩個 translation block 串接起來。這個就叫做 block chaining/linking。

在 QEMU 中，達成 block chaining 有兩種做法: 第一，採用 direct jump。此法直接修改 code cache 中分支指令的跳躍目標，因此依據 host 有不同的 patch 方式。第二，則是透過修改 TranslationBlock 的 tb\_next 欄位達成 block chaining。exec-all.h 中定義那些 host 支援使用 direct jump。

這裡只介紹 direct jump。我們先回到 cpu\_exec (cpu-exec.c) 的內層迴圈。

tb \= tb\_find\_fast(env);
 
if (tb\_invalidated\_flag) {
    next\_tb \= 0; // 注意! next\_tb 也被用來控制是否要做 block chaining。
    tb\_invalidated\_flag \= 0;
}
 
// 注意!! next\_tb 的名字會讓人誤解。block chaining 的方向為: next\_tb -> tb。
// next\_tb 不為 NULL 且 tb (guest binary) 不跨 guest page 的話，做 block
// chaining。原因之後再講。
if (next\_tb != 0 && tb\->page\_addr\[1\] \== \-1) {
    // 這邊利用 TranlationBlock 指針的最低有效位後兩位指引 block chaining 的方向。
    tb\_add\_jump((TranslationBlock \*)(next\_tb & ~3), next\_tb & 3, tb);
}
 
// 執行 TB，也就是 tc\_ptr 所指到的位址。注意，產生 TCG IR 的過程中，在 block 的最後會是
// exit\_tb addr，此 addr 是正在執行的 TranslationBlock 指針，同時也是 tcg\_qemu\_tb\_exec
// 的回傳值。該位址後兩位會被填入 0、1 或 2 以指示 block chaining 的方向。
next\_tb \= tcg\_qemu\_tb\_exec(tc\_ptr);

是不是有點暈頭轉向? 我們來仔細檢驗 guest binary -> TCG IR -> host binary 到底怎麼做的。在一個 translation block 的結尾，TCG 都會產生 TCG IR `exit_tb`。我們來看看。:-)

- tcg\_gen\_exit\_tb (tcg/tcg-op.h) 呼叫 tcg\_gen\_op1i 生成 TCG IR，其 opcode 為 INDEX\_op\_exit\_tb (還記得 `tcg.i` 裡的 TCGOpcode 嗎?)，operand 為 val。

// 一般是這樣呼叫: tcg\_gen\_exit\_tb((tcg\_target\_long)tb + tb\_num);
// 注意! 請留意其參數: (tcg\_target\_long)tb + tb\_num。
static inline void tcg\_gen\_exit\_tb(tcg\_target\_long val)
{
    // 將 INDEX\_op\_exit\_tb 寫入 gen\_opc\_buf; val 寫入 gen\_opparam\_buf。
    tcg\_gen\_op1i(INDEX\_op\_exit\_tb, val);
}

- tcg/ARCH/tcg-target.c 根據 TCG IR 產生對應 host binary。以 i386 為例: (關於每一行的作用，我是憑經驗用猜的。如有錯請指正。)

static inline void tcg\_out\_op(TCGContext \*s, int opc,
                              const TCGArg \*args, const int \*const\_args)
{
    // 總和效果: 返回 QEMU。
    //           next\_tb = tcg\_qemu\_tb\_exec(tc\_ptr);
    //
    // 圖示:
    // struct TranslationBlock\*         code cache
    //        next\_tb->tc\_ptr    ->        tb
    //
    // next\_tb 的末兩位編碼 next\_tb 條件分支的方向。
    // 等待下一次迴圈取得 tb = tb\_find\_fast()，
    // 試圖做 block chaining: next\_tb -> tb 
    //
    case INDEX\_op\_exit\_tb:
        // QEMU 把跳至 code cache 執行當作是函式呼叫，EAX 存放返回值。
        // 將 val 寫進 EAX，val 是 (tcg\_target\_long)tb + tb\_num。 
        tcg\_out\_movi(s, TCG\_TYPE\_I32, TCG\_REG\_EAX, args\[0\]);
 
        // e9 是 jmp 指令，後面的 operand 為相對偏移量，將會加上 eip。
        // 底下兩條的總和效果是跳回 code\_gen\_prologue 中 prologue 以後的位置。
        tcg\_out8(s, 0xe9); /\* jmp tb\_ret\_addr \*/
        // tb\_ret\_addr 在 tcg\_target\_qemu\_prologue 初始成指向
        // code\_gen\_prologue 中 prologue 以後的位置。
        // 生成 host binary 的同時，s->code\_ptr 會移向下一個 code buffer 的位址。
        // 所以要減去 4。
        tcg\_out32(s, tb\_ret\_addr \- s\->code\_ptr \- 4);
        break;
}

- tcg\_out\_movi 將 arg 移至 ret 代表的暫存器。

static inline void tcg\_out\_movi(TCGContext \*s, TCGType type,
                                int ret, int32\_t arg)
{
    if (arg \== 0) {
        /\* xor r0,r0 \*/
        tcg\_out\_modrm(s, 0x01 | (ARITH\_XOR << 3), ret, ret);
    } else {
        // move arg 至 ret
        // 0xb8 為 move，ret 代表目地暫存器。
        // 0xb8 + ret 合成一個 opcode。
        tcg\_out8(s, 0xb8 + ret);
        tcg\_out32(s, arg);
    }
}

小結一下，QEMU 函式名稱命名慣例為:

- tcg\_gen\_xxx – 產生 TCG Opcode 和 operand 至 gen\_opc\_buf 和 gen\_opparam\_buf。
- tcg\_out\_xxx – 產生 host binary 至 TCGContext 所指的 code cache。
