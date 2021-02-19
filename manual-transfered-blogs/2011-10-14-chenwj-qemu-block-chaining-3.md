原始博客链接：

转换时间：
2021-01-31

注意：由于2011年HelloGCC博客遭遇了一次VPS掉电丢失，所有2011年之前（含）的博客都是从 Google Reader 等RSS中恢复的。损失了大部分的内容，非常的遗憾。如果你的电脑中保存过HelloGCC2012年之前的报告，欢迎发送给我们，非常感谢。请转发给 hellogcc.workgroup@gmail.com

# QEMU Internal – Block Chaining 3/3
Posted on 2011/10/14 by chenwj

Copyright (c) 2011 陳韋任 (Chen Wei-Ren)

2. Block Chaining

由 guest binary -> TCG IR 的過程中，gen_goto_tb 會做 block chaining 的準備。 我們先來看何時會呼叫到 gen_goto_tb。以 i386 為例，遇到 guest binary 中的條件分支和直接跳轉都會呼叫 gen_goto_tb (target-i386/translate.c)。這邊以條件分支當例子:
```
static inline void gen_jcc(DisasContext *s, int b,
                           target_ulong val, target_ulong next_eip)
{
    int l1, l2, cc_op;

    cc_op = s->cc_op;
    gen_update_cc_op(s);
    if (s->jmp_opt) { // use direct block chaining for direct jumps
        l1 = gen_new_label();
        gen_jcc1(s, cc_op, b, l1);

        gen_goto_tb(s, 0, next_eip); // 我猜是 taken

        gen_set_label(l1);
        gen_goto_tb(s, 1, val); // 我猜是 not taken
        s->is_jmp = DISAS_TB_JUMP;
    } else {

      /* 忽略不提 */

    }
}
```
- gen_goto_tb。強烈建議閱讀 Porting QEMU to Plan 9: QEMU Internals and Port Strategy 2.2.3 和 2.2.4 節，也別忘了 SOURCEARCHIVE.com。
```
    // tb_num 代表目前 tb block linking 分支情況。eip 代表跳轉目標。
    static inline void gen_goto_tb(DisasContext *s, int tb_num, target_ulong eip)
    {
        TranslationBlock *tb;
        target_ulong pc;

        // s->pc 代表翻譯至目前 guest binary 的所在位址。tb->pc 表示 guest binary 的起始位址。
        // 注意! 這裡 s->cs_base + eip 代表跳轉位址; s->pc 代表目前翻譯到的 guest pc。
        pc = s->cs_base + eip; // 計算跳轉目標的 pc
        tb = s->tb; // 目前 tb
        // http://lists.nongnu.org/archive/html/qemu-devel/2011-08/msg02249.html
        // http://lists.gnu.org/archive/html/qemu-devel/2011-09/msg03065.html
        // 滿足底下兩個條件之一，則可以做 direct block linking
        // 第一，跳轉目標和目前 tb 起始的 pc 同屬一個 guest page。
        // 第二，跳轉目標和目前翻譯到的 pc 同屬一個 guest page。
        if ((pc & TARGET_PAGE_MASK) == (tb->pc & TARGET_PAGE_MASK) ||
            (pc & TARGET_PAGE_MASK) == ((s->pc - 1) & TARGET_PAGE_MASK))  {
            // 如果 guest jump 指令和其跳轉位址同屬一個 guest page，則做 direct block linking。

            tcg_gen_goto_tb(tb_num); // 生成準備做 block linking 的 TCG IR。詳情請見之後描述。

            // 更新 env 的 eip，使其指向此 tb 之後欲執行指令的位址。
            // tb_find_fast 會用 eip 查找該 TB 是否已被翻譯過。
            gen_jmp_im(eip);

            // 最終回到 QEMU tcg_qemu_tb_exec，賦值給 next_tb。
            // 注意! tb_num 會被 next_tb & 3 取出，由此可以得知 block chaining 的方向。
            tcg_gen_exit_tb((tcg_target_long)tb + tb_num);
        } else {
            /* jump to another page: currently not optimized */
            gen_jmp_im(eip);
            gen_eob(s);
        }
    }
```
- tcg_gen_goto_tb 生成 TCG IR。
```
        static inline void tcg_gen_goto_tb(int idx)
        {
            tcg_gen_op1i(INDEX_op_goto_tb, idx);
        }

        tcg_out_op (tcg/i386/tcg-target.c) 將 TCG IR 翻成 host binary。注意! 這邊利用 patch jmp 跳轉位址達成 block linking。

        static inline void tcg_out_op(TCGContext *s, TCGOpcode opc,
                                      const TCGArg *args, const int *const_args)
        {
            case INDEX_op_goto_tb:
                if (s->tb_jmp_offset) {
                    /* direct jump method */
                    tcg_out8(s, OPC_JMP_long); /* jmp im */
                    // 紀錄將來要 patch 的地方。
                    s->tb_jmp_offset[args[0]] = s->code_ptr - s->code_buf;
                    // jmp 的參數為 jmp 下一個指令與目標的偏移量。
                    // 如果還沒做 block chaining，則 jmp 0 代表 fall through。
                    tcg_out32(s, 0);
                } else {

                    /* 在此忽略 */

                }
                // 留待將來 "reset" direct jump 之用。
                s->tb_next_offset[args[0]] = s->code_ptr - s->code_buf;
                break;
        }
```
回答上一篇最後留下的問題。在還未 patch code cache 中的分支跳轉指令的跳轉位址，它會 fall through，還記得 jmp 0 嗎? 我這邊在列出 gen_goto_tb 的部分內容:
```
tcg_gen_goto_tb(tb_num);

// Fall through

// 更新 env 的 eip，使其指向此 tb 之後欲執行指令的位址。
// tb_find_fast 會用 eip 查找該 TB 是否已被翻譯過。
gen_jmp_im(eip);

// 最終回到 QEMU tcg_qemu_tb_exec，賦值給 next_tb。
// 注意! tb_num 會被 next_tb & 3 取出，由此可以得知 block chaining 的方向。
tcg_gen_exit_tb((tcg_target_long)tb + tb_num);
```
目前執行的 tb 會賦值給 next_tb (末兩位編碼 block chaining 的方向)。等待下一次迴圈，tb_find_fast 回傳 next_tb 的下一個 tb。
```
if (next_tb != 0 && tb->page_addr[1] == -1) {
    // 這邊利用 TranlationBlock 指針的最低有效位後兩位指引 block chaining 的方向。
    // next_tb -> tb
    tb_add_jump((TranslationBlock *)(next_tb & ~3), next_tb & 3, tb);
}
```
That’s it! That’s how direct block chaining is done in QEMU, I think…
