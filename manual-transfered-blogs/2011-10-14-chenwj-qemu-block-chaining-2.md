原始博客链接：

转换时间：
2021-01-31

注意：由于2011年HelloGCC博客遭遇了一次VPS掉电丢失，所有2011年之前（含）的博客都是从 Google Reader 等RSS中恢复的。损失了大部分的内容，非常的遗憾。如果你的电脑中保存过HelloGCC2012年之前的报告，欢迎发送给我们，非常感谢。请转发给 hellogcc.workgroup@gmail.com

# QEMU Internal – Block Chaining 2/3
Posted on 2011/10/14 by chenwj

Copyright (c) 2011 陳韋任 (Chen Wei-Ren)

2. Block Chaining

我們再回到 cpu_exec (cpu-exec.c) 的內層迴圈。這邊主要看 tb_add_jump。
```
if (next_tb != 0 && tb->page_addr[1] == -1) {
    // 這邊利用 TranlationBlock 指針的最低有效位後兩位指引 block chaining 的方向。
    // next_tb -> tb
    tb_add_jump((TranslationBlock *)(next_tb & ~3), next_tb & 3, tb);
}

// 這邊要注意到，QEMU 利用 TranslatonBlock 指針後兩位必為零的結果
// 做了一些手腳。QEMU 將其末兩位編碼成 0、1 或 2 來指引 block chaing
// 的方向。這種技巧在 QEMU 用得非常嫻熟。
next_tb = tcg_qemu_tb_exec(tc_ptr);
```
-    tb_add_jump 呼叫 tb_set_jmp_target 做 block chaining 的 patch。另外，會利用 tb 的 jmp_next 和 tb_next 的 jmp_first 把 block chaining 中的 TranslationBlock 串成一個 circular list。這是以後做 block *unchaining* 之用。再複習一下 TranslationBlock，
```
        struct TranslationBlock*        code cache (host binary)
                tb->tc_ptr        ->            TB
```
    請注意! 當我們講 TB，可能是指 TranslationBlock，也可能是指 code cache 中的 TB。好! 當我們 patch code cache 中的 TB，將它們串接在一起。我們同時也要把它們相對應的 TranslationBlock 做點紀錄，利用 jmp_first 和 jmp_next 這兩個欄位。

    這邊我先回到 tb_link_page，還記得我留下一些地方沒講嘛? 開始吧! 😉
```
    // jmp_first 代表跳至此 tb 的其它 TB 中的頭一個。jmp_first 初值為自己，末兩位為 10 (2)。
    // 將來做 block chaining 時，jmp_first 指向跳至此 tb 的其它 TB 中的頭一個，tb1，末兩位
    // 為 00 或 01，這代表從 tb1 的哪一個分支跳至此 tb。
    tb->jmp_first = (TranslationBlock *)((long)tb | 2);

    // jmp_next[n] 代表此 tb 條件分支的目標 TB。
    // 注意! 如果目標 TB，tb2，孤身一人，jmp_next 就真的指向 tb2 (符合初次看到 jmp_next 所想
    // 的語意)。
    //
    // 如果已有其它 TB，tb3，跳至 tb2，則賦值給 tb->jmp_next 的是 tb2 的 jmp_first，也就是
    // tb3 (末兩位編碼 tb3 跳至 tb2 的方向)。
    tb->jmp_next[0] = NULL;
    tb->jmp_next[1] = NULL;

    // tb_next_offset 代表此 TB 在 code cache 中分支跳轉要被 patch 的位址 (相對於其 code cache
    // 的偏移量)，為了 direct block chaining 之用。
    if (tb->tb_next_offset[0] != 0xffff)
        tb_reset_jump(tb, 0);
    if (tb->tb_next_offset[1] != 0xffff)
        tb_reset_jump(tb, 1);
```
    我們再回來看看 tb_add_jump 做了什麼。:-)
```
    // block chaining 方向為: tb -> tb_next。n 用來指示 tb 條件分支的方向。
    static inline void tb_add_jump(TranslationBlock *tb, int n,
                                   TranslationBlock *tb_next)
    {
        // jmp_next[0]/jmp_next[1] 代表 tb 條件分支的目標。
        if (!tb->jmp_next[n]) {
            /* patch the native jump address */
            tb_set_jmp_target(tb, n, (unsigned long)tb_next->tc_ptr);

            // tb_jmp_remove 會用到 jmp_next 做 block unchaining，這個以後再講。

            // tb_next->jmp_first 初值為自己，末兩位設為 10 (2)。
            // 如果已有其它 TB，tb1，跳至 tb_next，則下面這條語句會使得
            // tb->jmp_next 指向 tb1 (末兩位代表 tb1 跳至 tb_next 的方向)。
            // tb_next->jmp_first 由原本指向 tb1 改指向 tb。
            // 有沒有 circular list 浮現在你腦海中? ;-)
            tb->jmp_next[n] = tb_next->jmp_first;

            // tb_next 的 jmp_first 指回 tb，末兩位代表由 tb 哪一個條件分支跳至 tb_next。
            tb_next->jmp_first = (TranslationBlock *)((long)(tb) | (n));
        }
    }
```
我們再來看是怎麼 patch code cache 中分支指令的目標地址。依據是否採用 direct jump，tb_set_jmp_target (exec-all.h) 有不同做法。採用 direct jump 的話，tb_set_jmp_target 會根據 host 呼叫不同的 tb_set_jmp_target1。tb_set_jmp_target1 會用到 TB 的 tb_jmp_offset。如果不採用 direct jump 做 block chaining，tb_set_jmp_target 會直接修改 TB 的 tb_next。

-    tb_set_jmp_target (exec-all.h)。
```
    static inline void tb_set_jmp_target(TranslationBlock *tb,
                                         int n, unsigned long addr)
    {
        unsigned long offset;

        // n 可以為 0 或 1，代表分支跳轉的分向 taken 或 not taken。
        offset = tb->tb_jmp_offset[n]; // tb 要 patch 的位址相對於 tb->tc_ptr 的偏移量。
        tb_set_jmp_target1((unsigned long)(tb->tc_ptr + offset), addr);
    }
```
-    tb_set_jmp_target1 (exec-all.h)。
```
    #elif defined(__i386__) || defined(__x86_64__)
    static inline void tb_set_jmp_target1(unsigned long jmp_addr, unsigned long addr)
    {
        /* patch the branch destination */
        // jmp 的參數為 jmp 下一條指令與目標地址的偏移量。
        *(uint32_t *)jmp_addr = addr - (jmp_addr + 4);
        /* no need to flush icache explicitly */
    }
```
你會有這個疑問嗎? 在分支跳轉位址被 patch 到分支跳轉指令之前，它是要跳去哪裡? 🙂

[QEMU – block chaining](http://www.hellogcc.org/wp-content/uploads/2011/10/QEMU-block-chaining.ppt) 可以幫助你比較清楚了解 block chaining 如何運作。
