原始博客链接：

转换时间：
2021-01-31

注意：由于2011年HelloGCC博客遭遇了一次VPS掉电丢失，所有2011年之前（含）的博客都是从 Google Reader 等RSS中恢复的。损失了大部分的内容，非常的遗憾。如果你的电脑中保存过HelloGCC2012年之前的报告，欢迎发送给我们，非常感谢。请转发给 hellogcc.workgroup@gmail.com

# QEMU Internal – Block Chaining 1/3
Posted on 2011/10/14 by chenwj

Copyright (c) 2011 陳韋任 (Chen Wei-Ren)

2. Block Chaining

先複習一下 QEMU 的流程，目前的情況如底下這樣:
```
    QEMU -> code cache -> QEMU -> code cache -> ...
```
之前提到，QEMU 是以一個 translation block 為單位進行翻譯並執行。這也就是說，每當在 code cache 執行完一個 translation block 之後，控制權必須交還給 QEMU。這很沒有效率。理想情況下，大部分時間應該花在 code cache 中執行，只要在必要情況下才需要回到 QEMU。基本想法是把 code cache 中的 translation block 串接起來。只要 translation block 執行完之後，它的跳躍目標確定且該跳躍目標也已經在 code cache 裡，那我們就把這兩個 translation block 串接起來。這個就叫做 block chaining/linking。

在 QEMU 中，達成 block chaining 有兩種做法: 第一，採用 direct jump。此法直接修改 code cache 中分支指令的跳躍目標，因此依據 host 有不同的 patch 方式。第二，則是透過修改 TranslationBlock 的 tb_next 欄位達成 block chaining。exec-all.h 中定義那些 host 支援使用 direct jump。

這裡只介紹 direct jump。我們先回到 cpu_exec (cpu-exec.c) 的內層迴圈。
```
tb = tb_find_fast(env);

if (tb_invalidated_flag) {
    next_tb = 0; // 注意! next_tb 也被用來控制是否要做 block chaining。
    tb_invalidated_flag = 0;
}

// 注意!! next_tb 的名字會讓人誤解。block chaining 的方向為: next_tb -> tb。
// next_tb 不為 NULL 且 tb (guest binary) 不跨 guest page 的話，做 block
// chaining。原因之後再講。
if (next_tb != 0 && tb->page_addr[1] == -1) {
    // 這邊利用 TranlationBlock 指針的最低有效位後兩位指引 block chaining 的方向。
    tb_add_jump((TranslationBlock *)(next_tb & ~3), next_tb & 3, tb);
}

// 執行 TB，也就是 tc_ptr 所指到的位址。注意，產生 TCG IR 的過程中，在 block 的最後會是
// exit_tb addr，此 addr 是正在執行的 TranslationBlock 指針，同時也是 tcg_qemu_tb_exec
// 的回傳值。該位址後兩位會被填入 0、1 或 2 以指示 block chaining 的方向。
next_tb = tcg_qemu_tb_exec(tc_ptr);
```
是不是有點暈頭轉向? 我們來仔細檢驗 `guest binary -> TCG IR -> host binary` 到底怎麼做的。在一個 translation block 的結尾，TCG 都會產生 TCG IR exit_tb。我們來看看。:-)
```
    tcg_gen_exit_tb (tcg/tcg-op.h) 呼叫 tcg_gen_op1i 生成 TCG IR，其 opcode 為 INDEX_op_exit_tb (還記得 tcg.i 裡的 TCGOpcode 嗎?)，operand 為 val。

    // 一般是這樣呼叫: tcg_gen_exit_tb((tcg_target_long)tb + tb_num);
    // 注意! 請留意其參數: (tcg_target_long)tb + tb_num。
    static inline void tcg_gen_exit_tb(tcg_target_long val)
    {
        // 將 INDEX_op_exit_tb 寫入 gen_opc_buf; val 寫入 gen_opparam_buf。
        tcg_gen_op1i(INDEX_op_exit_tb, val);
    }
```
- tcg/ARCH/tcg-target.c 根據 TCG IR 產生對應 host binary。以 i386 為例: (關於每一行的作用，我是憑經驗用猜的。如有錯請指正。)
```
    static inline void tcg_out_op(TCGContext *s, int opc,
                                  const TCGArg *args, const int *const_args)
    {
        // 總和效果: 返回 QEMU。
        //           next_tb = tcg_qemu_tb_exec(tc_ptr);
        //
        // 圖示:
        // struct TranslationBlock*         code cache
        //        next_tb->tc_ptr    ->        tb
        //
        // next_tb 的末兩位編碼 next_tb 條件分支的方向。
        // 等待下一次迴圈取得 tb = tb_find_fast()，
        // 試圖做 block chaining: next_tb -> tb
        //
        case INDEX_op_exit_tb:
            // QEMU 把跳至 code cache 執行當作是函式呼叫，EAX 存放返回值。
            // 將 val 寫進 EAX，val 是 (tcg_target_long)tb + tb_num。
            tcg_out_movi(s, TCG_TYPE_I32, TCG_REG_EAX, args[0]);

            // e9 是 jmp 指令，後面的 operand 為相對偏移量，將會加上 eip。
            // 底下兩條的總和效果是跳回 code_gen_prologue 中 prologue 以後的位置。
            tcg_out8(s, 0xe9); /* jmp tb_ret_addr */
            // tb_ret_addr 在 tcg_target_qemu_prologue 初始成指向
            // code_gen_prologue 中 prologue 以後的位置。
            // 生成 host binary 的同時，s->code_ptr 會移向下一個 code buffer 的位址。
            // 所以要減去 4。
            tcg_out32(s, tb_ret_addr - s->code_ptr - 4);
            break;
    }
```
- tcg_out_movi 將 arg 移至 ret 代表的暫存器。
```
        static inline void tcg_out_movi(TCGContext *s, TCGType type,
                                        int ret, int32_t arg)
        {
            if (arg == 0) {
                /* xor r0,r0 */
                tcg_out_modrm(s, 0x01 | (ARITH_XOR << 3), ret, ret);
            } else {
                // move arg 至 ret
                // 0xb8 為 move，ret 代表目地暫存器。
                // 0xb8 + ret 合成一個 opcode。
                tcg_out8(s, 0xb8 + ret);
                tcg_out32(s, arg);
            }
        }
```
小結一下，QEMU 函式名稱命名慣例為:

- tcg_gen_xxx – 產生 TCG Opcode 和 operand 至 gen_opc_buf 和 gen_opparam_buf。
- tcg_out_xxx – 產生 host binary 至 TCGContext 所指的 code cache。
