---
title: "QEMU Internal – Precise Exception Handling 3/5"
date: "2012-03-14"
categories: 
  - "qemu"
---

Copyright (c) 2012 陳韋任 (Chen Wei-Ren)

首先我們來看 \_\_stb\_mmu。請看 ${SRC}/softmmu\_template.h 和 ${BUILD}/i386-softmmu/op\_helper.i。SOFTMMU 相關的 helper function 是透過 softmmu\_\* 檔案內的巨集加以合成。這裡只挑部分加以描述。

SUFFIX 可以是 b (byte, 8)、w (word, 16)、l (long word, 32) 和 q (quad word, 64)，代表資料大小。MMUSUFFIX 可以是 cmmu 或是 mmu，分別代表欲讀取的是 code 或是 data。mmu\_idx 代表索引的是內核態亦或是用戶態的 TLB。addr 代表 guest virtual address。

// ${SRC}/softmmu\_template.h
void REGPARM glue(glue(\_\_st, SUFFIX), MMUSUFFIX)(target\_ulong addr,
                                                 DATA\_TYPE val,
                                                 int mmu\_idx)
{
   ...
}

接著看展開巨集後的函式體。

// ${BUILD}/i386-softmmu/op\_helper.i
void \_\_stb\_mmu(target\_ulong addr, uint8\_t val, int mmu\_idx)
{
 redo:
    // 查找 TLB
    tlb\_addr \= env\-&gt;tlb\_table\[mmu\_idx\]\[index\].addr\_write;
    if (...) {
 
        // TLB 命中
 
    } else {
 
        // TLB 不命中
 
        /\* the page is not in the TLB : fill it \*/
        // retaddr = GETPC();
        retaddr \= ((void \*)((unsigned long)\_\_builtin\_return\_address(0) \- 1));
 
        // 試圖填入 TLB entry。
        tlb\_fill(addr, 1, mmu\_idx, retaddr);
        goto redo;
    }
}

這裡 QEMU 利用 GCC 的 \_\_builtin\_return\_address 擴展 \[1\] 來判定 tlb\_fill 是從一般 C 函式或是 code cache 中被呼叫。retaddr 若為 0，表前者，retaddr 若不為 0，表後者。之後，我們會透過 GDB 更加清楚前面所述所代表的意思。我們關注 retaddr 不為 0，也就是從 code cache 中呼叫 tlb\_fill 的情況。

在看 tlb\_fill 之前，我們先偷看 cpu\_x86\_handle\_mmu\_fault (target-i386/helper.c) 的註解。我們關注返回值為 1，也就是頁缺失的情況。

/\* return value:
   -1 = cannot handle fault
   0  = nothing more to do
   1  = generate PF fault
\*/
int cpu\_x86\_handle\_mmu\_fault(CPUX86State \*env, target\_ulong addr, ...)
{
  ...
}

我們來看 tlb\_fill。

void tlb\_fill(target\_ulong addr, int is\_write, int mmu\_idx, void \*retaddr)
{
    ret \= cpu\_x86\_handle\_mmu\_fault(env, addr, is\_write, mmu\_idx, 1);
    if (ret) {
        if (retaddr) {
 
            // 當客戶發生頁缺失 (ret == 1) 且 tlb\_fill 是從 code cache 中被
            // 呼叫 (retaddr != 0)，我們會在這裡。
 
            /\* now we have a real cpu fault \*/
            pc \= (unsigned long)retaddr;
            tb \= tb\_find\_pc(pc);
            if (tb) {
                /\* the PC is inside the translated code. It means that we have
                   a virtual CPU fault \*/
                cpu\_restore\_state(tb, env, pc, NULL);
            }
        }
        raise\_exception\_err(env\-&gt;exception\_index, env\-&gt;error\_code);
    }
    env \= saved\_env;
}

請注意! 如果 retaddr != 0，其值代表的 (幾乎) 是發生例外的 host binary 所在位址。QEMU 利用它來查找是哪一個 TranslationBlock 中的 host binary 發生例外。tb\_find\_pc (exec.c) 利用該 host binary pc 進行查找，取得 tb。

TranslationBlock \*tb\_find\_pc(unsigned long tc\_ptr)
{
    // tbs 是 TranslationBlock \* 數組。每一個在 code cache 中 (已翻譯好的)
    // basic block 都有相對應的 TranslationBlock 存放其相關資訊。
 
    /\* binary search (cf Knuth) \*/
    m\_min \= 0;
    m\_max \= nb\_tbs \- 1;
    while (m\_min &gt; 1;
        tb \= &amp;tbs\[m\];
        // tc\_ptr 代表 host binary 在 code cache 的起始位址。
        v \= (unsigned long)tb\-&gt;tc\_ptr;
        if (v \== tc\_ptr)
            return tb;
        else if (tc\_ptr &lt; v) {
            m\_max \= m \- 1;
        } else {
            m\_min \= m + 1;
        }
    }
    return &amp;tbs\[m\_max\];
}

一但找到該負責的 tb，QEMU 就會回復 guest CPUState 以便 guest exception handler 處理 guest 的頁缺失例外。

    if (tb) {
        /\* the PC is inside the translated code. It means that we have
           a virtual CPU fault \*/
        cpu\_restore\_state(tb, env, pc, NULL);
    }

接著我們看 cpu\_restore\_state (translate-all.c)。

\[1\] http://gcc.gnu.org/onlinedocs/gcc/Return-Address.html
