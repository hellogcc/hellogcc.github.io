原始博客链接：

转换时间：
2021-01-31

注意：由于2011年HelloGCC博客遭遇了一次VPS掉电丢失，所有2011年之前（含）的博客都是从 Google Reader 等RSS中恢复的。损失了大部分的内容，非常的遗憾。如果你的电脑中保存过HelloGCC2012年之前的报告，欢迎发送给我们，非常感谢。请转发给 hellogcc.workgroup@gmail.com

# QEMU Internal – Tiny Code Generator (TCG) 1/2
Posted on 2011/10/09 by chenwj

Copyright (c) 2011 陳韋任 (Chen Wei-Ren)

前言

因為工作上的關係，必須接觸 QEMU。雖然網路上有不少文件，但總覺得講得不夠深入。QEMU 是一個仿真器 (emulator)，可以 process mode 或是 system mode 運行。process mode 可以運行不同 ISA 同一 OS 的 binary; system mode 可以在當前作業系統上運行另外一個 OS。我在收集各方資料，閱讀代碼和在郵件列表上發問之後，覺得略有心得。在此對 QEMU internal 作一個較為深入的介紹。憑我個人之力，難免有疏漏或是錯誤。權且當作拋磚引玉吧。希望各位不吝指教。

0. 術語、線上資源和技巧

對 QEMU 而言，被仿真的平台被稱為 guest，又稱 target; 運行 QEMU 的平台稱為 host。QEMU 是利用動態翻譯 (dynamic translation) 的技術將 guest binary 動態翻譯成 host binary，並交由 host 運行翻譯所得的 host binary。Tiny Code Generator (TCG) 是 QEMU 中負責動態翻譯的組件。對 TCG 而言，target 有不同的含意，它代表 TCG 是針對哪一個 host 生成 host binary。

網路上對 QEMU 有較為完整描述的文件為:

```    QEMU, a Fast and Portable Dynamic Translator
    Porting QEMU to Plan 9: QEMU Internals and Port Strategy
```
然而需要注意的是，上述文件在動態翻譯的部分均是針對 QEMU 0.9 版。QEMU 0.9 版以前是使用 dyngen 技術; QEMU 0.10 版以後採用 TCG。雖說如此，但在 QEMU 的其它部分差異不大，上述文件仍可供參考。SOURCEARCHIVE.com 收集了自 QEMU 0.6.1 版至今的所有 QEMU 源代碼。各位可以邊看文件邊看源代碼。

QEMU 極為依賴 macro，這使得直接閱讀源代碼通常無法確定其函數呼叫，或是執行流程倒底為何。請在編譯 QEMU 的時候加上 `--extra-cflags="-save-temps"`，如此可得展開 marco 的 `*.i `檔。

其餘部分請見:

```    Getting started for developers
    QEMU Internals
    QEMU 目錄下的 HACKING、CODING_STYLE、tcg/README 和 doc/*
    ISA reference manual。由於本篇文章是以 x86 為例，關於 x86 指令請見  x86/x64 指令编码内幕。
```
1. TCG

TCG 是 QEMU 的核心。其基本流程如下:

`guest binary -> TCG IR -> host binary`

1.1 TCG IR

TCG 定義了一組 IR (intermediate representation)，熟悉 GCC 的各位對此應該不陌生。TCG IR 大致分成以下幾類:

```    Move Operation: mov, movi, …
    Logic Operation: and, or, xor, shl, shr, …
    Arithmetic peration: add, sub, mul, div, …
    Branch Operation: jmp, br, brcond
    Fuction call: call
    Memory Operation: ld, st
    QEMU specific Operation: tb_exit, goto_tb, qemu_ld/qemu_st
    ```

請見 `tcg/*`，特別是 tcg.i，可以看到 `TCGOpcode`。`tcg/README` 也別忘了。TCG 在翻譯 guest binary 的時候是以一個 `translation block (tb)` 為單位，其結尾通常是分支指令。

`target-ARCH/*` 定義了如何將` ARCH binary` 反匯編成 TCG IR。tcg/ARCH 定義了如何將 TCG IR 翻譯成 ARCH binary。

1.2 TCG Flow

先介紹一些資料結構:
```
    gen_opc_buf 和 gen_opparam_buf (translate-all.c) 分別放置 TCG Opcode 和 Operand。
    如果使用靜態配置的緩衝區，static_code_gen_buffer (exec.c) 即為 code cache，放置 host binary。
    在跳入/出 code cache 執行之前/後，要執行 prologue/epilogue，請見 code_gen_prologue (exec.c)。這邊的 prologue/epilogue 就是指 function prologue/epilogue。QEMU 將跳至 code cache (host binary) 執行的過程看成是函式呼叫，故有此 prologue/epilogue。
```
以 qemu-i386 為例，流程大致如下:
```
main (linux-user/main.c) -> cpu_exec_init_all (exec.c)
  -> cpu_init/cpu_x86_init (target-i386/helper.c)
  -> tcg_prologue_init (tcg/tcg.c) -> cpu_loop (linux-user/main.c)
```
函式名之所以會出現 cpu_init/cpu_x86_init，是因為 QEMU 經常使用 #define 替換函式名。cpu_init 是 main 裡呼叫的函式，經 #define 替換後，實際上是 cpu_x86_init (target-i386/helper.c)。GDB 下斷點時請注意此種情況。

這邊只介紹 tcg_prologue_init (tcg/tcg.c) -> cpu_loop (linux-user/main.c) 這一段，因為這一段跟 TCG 較為相關。容我先講 cpu_loop (linux-user/main.c)。
```
    cpu_loop (linux-user/main.c) -> cpu_x86_exec/cpu_exec (cpu-exec.c)。cpu_exec 是主要執行迴圈，其結構大致如下:

    /* prepare setjmp context for exception handling */
    for(;;) {
        if (setjmp(env->jmp_env) == 0) { // 例外處理。
        }

        next_tb = 0; /* force lookup of first TB */
        for(;;) {
          // 判斷是否有中斷。若有，跳回例外處理。

          next_tb = tcg_qemu_tb_exec(tc_ptr); // 跳至 code cache 執行。

        }
    }
```
    tcg_prologue_init (tcg/tcg.c) -> tcg_target_qemu_prologue (tcg/i386/tcg-target.c)。如前所述，QEMU 將跳至 code cache (host binary) 執行的過程看成是函式呼叫。不同平台的 calling convention 各有不同，tcg_prologue_init 將產生 prologue/epilogue 的工作轉交 tcg_target_qemu_prologue。
```
    static void tcg_target_qemu_prologue(TCGContext *s)
    {
      /* QEMU (cpu_exec) -> 入棧 */

      // OPC_GRP5 (0xff) 為 call，EXT5_JMPN_Ev 是其 opcode extension。
      // tcg_target_call_iarg_regs 是函式呼叫負責傳遞參數的暫存器。
      // 跳至 code cache 執行。
      tcg_out_modrm(s, OPC_GRP5, EXT5_JMPN_Ev, tcg_target_call_iarg_regs[0]);

      // 此時，s->code_ptr 指向 code_gen_prologue 中 prologue 和 jmp to code cache
      // 之後的位址。
      // tb_ret_addr 是紀錄 code cache 跳回 code_gen_prologue 的哪個地方。
      tb_ret_addr = s->code_ptr;

      /* 出棧 -> 返回 QEMU (cpu_exec)，確切的講是返回 tcg_qemu_tb_exec */
    }
```
這邊小結一下。

```QEMU -> prologue -> code cache -> epilogue -> QEMU
```
tb_ret_addr 就是用來由 code cache 返回至 code_gen_prologue，執行 epilogue，再返回 QEMU。

在介紹 cpu_exec 之前，我先介紹幾個 QEMU 資料結構，請善用 SOURCEARCHIVE.com。我們要知道所謂仿真或是虛擬化一個 CPU (ISA)，簡單來說就是用一個資料結構 (struct) 儲存該 CPU 的狀態。執行該虛擬 CPU，就是從內存中讀取該虛擬 CPU 的資料結構，運算後再存回去。

- CPUX86State: 保存 x86 register，eflags，eip，cs，…。不同 ISA 之間通用的資料結構被 QEMU #define 成 CPU_COMMON。一般稱此資料結構為 CPUState。下文所提 env 即為 CPUState。 QEMU 運行虛擬 CPU 都會利用 env 這個變數。
- TranslationBlock: 之前說過，QEMU 是以一個 translation block 為單位進行翻譯。其中保存此 translation block 對應 guest binary 的 pc, cs_base, eflags。另外，tc_ptr 指向 code cache (host binary)。其它欄位待以後再談。
- PageDesc: 主要保存 guest page 中的第一個 `tb (TranslationBlock *)`。這跟 QEMU 內部運作機制有關。某些情況下，guest page (guest binary) 可能被替換或是被寫。這個時候，QEMU 會以 guest page (guest binary) 為單位，清空與它相關聯的 TB (code cache)。這時再回來講 TranslationBlock。TranslationBlock 有底下兩個欄位:
  - page_addr[2]: 存放 TranslationBlock 對應 guest binary 所在的 guest page。注意! guest binary 有可能跨 guest page，故這裡有兩個欄位。
  - page_next[2]: 當透過 PageDesc->first_tb 找到該 guest page 的第一個 tb，tb->page_next 就被用來找尋該 guest page 的下一個 tb。

- 再回來講 PageDesc。QEMU 替 PageDesc 維護了一個二級頁表 l1_map。page_find 這個函式根據輸入的 address 搜尋 l1_map，返回 PageDesc。這在以 guest page (guest binary) 為單位，清空與它相關聯的 TB (code cache) 的時候會用到。有一個名字很像的資料結構叫 PhysPageDesc，QEMU 也替它維護一個二級頁表 l1_phys_map。這是在 system mode 做地址轉換之用，這邊不談。
- TCGContext: 生成 TCG IR 時會用到。
- DisasContext: 反匯編 guest binary 時會用到。

## 2 Comments

2012/08/02
r_buaa

陈先生，您好:
最近，我刚刚开始看QEMU代码，进度挺慢的，有几个问题想请教下：
1.镜像文件都是二进制的，是什么样的机制使得其能成为汇编后，然后再翻译成为host代码?
2.在翻译过程中在哪 部分代码是将镜像和tb连接起来的？这个我一直没找到
3.qemu虚拟的设备是在什么时候被用到？是在将guest代码翻译成为host代码时吗？
qemu底层的资料太少了，不知道陈先生您能不能给我点建议，如何去阅读？
谢谢


2012/08/17
chenwj

我建議您再讀讀 QEMU Internal – Tiny Code Generator (TCG) 2/2。我先試著回答您的問題。

1. gen_intermediate_code_internal (`target-*/translate.c`) 會呼叫 disas_insn 反組譯 guest binary 成 TCG IR。tcg_gen_code (tcg/tcg.c) 再將 TCG IR 轉成 host binary。

2. tb_gen_code (exec.c) 翻譯完 guest binary 之後，會有相對應的資料結構 TranslationBlock (tb)，不要跟 code cache 中的 tb 混在一起，記下 code cache 中的 tb 相對應 guest binary 的起始位址。你大致可以把它們三者想成底下這樣:

guest binary TranslationBlock (tb) code cache (tb)

3. 我之前有被問過類似問題，不知道你是不是就是寄信給我的那一位。你可以參考底下我回信給他的內容。:-)

http://people.cs.nctu.edu.tw/~chenwj/log/MAIL/gonewithyesterday-2012-08-06.txt
