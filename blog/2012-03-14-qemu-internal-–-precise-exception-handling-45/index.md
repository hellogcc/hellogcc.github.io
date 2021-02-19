---
title: "QEMU Internal – Precise Exception Handling 4/5"
date: "2012-03-14"
categories: 
  - "qemu"
---

Copyright (c) 2012 陳韋任 (Chen Wei-Ren)

好! 我們現在找到例外 (本範例是頁缺失) 是發生在某個 TranslationBlock 裡頭，但是到底是哪一條 guest 指令觸發頁缺失? 我們需要從頭翻譯該 TranslationBlock 對應的 guest binary 來揪出罪魁禍首。一般情況下，QEMU 在翻譯 guest binary 時不會記錄 guest pc 資訊。這時，為了定位 guest pc，QEMU 在翻譯 guest binary 會記錄額外的資訊，包含 guest pc。

QEMU 會用到底下定義在 translate-all.c 資料結構:

  target\_ulong gen\_opc\_pc\[OPC\_BUF\_SIZE\]; // 紀錄 guest pc。
  uint8\_t gen\_opc\_instr\_start\[OPC\_BUF\_SIZE\]; // 當作標記之用。

針對 x86，又在 target-i386/translate.c 定義以下資料結構:

  static uint8\_t gen\_opc\_cc\_op\[OPC\_BUF\_SIZE\]; // 紀錄 condition code。

現在來看 cpu\_restore\_state (translate-all.c)。searched\_pc 傳入的 (幾乎) 是發生例外的 host pc。

int cpu\_restore\_state(TranslationBlock \*tb,
                      CPUState \*env, unsigned long searched\_pc,
                      void \*puc)
{
    tcg\_func\_start(s); // 初始 gen\_opc\_ptr 和 gen\_opparam\_ptr
 
    // 轉呼叫 gen\_intermediate\_code\_internal，要求在生成 TCG IR
    // 的同時，為其生成相關的 guest pc 和其它資訊於下列資料結構。
    //
    //   gen\_opc\_pc, gen\_opc\_instr\_start, 和 gen\_opc\_cc\_op
    //
    gen\_intermediate\_code\_pc(env, tb);
 
    // 轉呼叫 tcg\_gen\_code\_common (tcg/tcg.c) 將 TCG IR 翻成 host binary。
    // 返回 TCG gen\_opc\_buf index。
    j \= tcg\_gen\_code\_search\_pc(s, (uint8\_t \*)tc\_ptr, searched\_pc \- tc\_ptr);
 
    // gen\_opc\_instr\_start\[j\] 若為 1，代表 gen\_opc\_pc\[j\] 和 gen\_opc\_cc\_op\[j\]
    // 正是我們所要的資訊。
    while (gen\_opc\_instr\_start\[j\] \== 0)
        j\--;
 
    // 回復 CPUState。
    gen\_pc\_load(env, tb, searched\_pc, j, puc);
 
}

gen\_intermediate\_code\_pc 是 gen\_intermediate\_code\_internal 的包裝，search\_pc 設為 1。當 search\_pc 為 true，在翻譯 guest binary 的同時，生成額外資訊。

static inline void gen\_intermediate\_code\_internal(CPUState \*env,
                                                  TranslationBlock \*tb,
                                                  int search\_pc)
{
    // guest binary -&gt; TCG IR
    for(;;) {
 
        if (search\_pc) {
            // gen\_opc\_ptr 為 TCG opcode buffer 目前位址，gen\_opc\_buf 為
            // TCG opcode buffer 的起始位址。
            j \= gen\_opc\_ptr \- gen\_opc\_buf;
            if (lj &lt; j) {
                lj++;
                while (lj cc\_op; // 紀錄 condition code。
            gen\_opc\_instr\_start\[lj\] \= 1; // 填 1 作為標記。
            gen\_opc\_icount\[lj\] \= num\_insns;
        }
 
        // 針對 pc\_ptr 代表的 guest pc 進行解碼並生成 TCG IR，返回下一個 guest pc。
        pc\_ptr \= disas\_insn(dc, pc\_ptr);
 
    }
}

tcg\_gen\_code\_search\_pc 是 tcg\_gen\_code\_common 的包裝，search\_pc (應命名為 offset) 設為發生例外的 host binary 與其所屬 basic block 在 code cache 開頭 (tc\_ptr) 的 offset。注意! 此時傳入 gen\_code\_buf 的是觸發例外的 TranslationBlock 其 tc\_ptr。也就是說，現在 TCG IR -> host binary 中的 host binary 是寫在發生例外 host binary 所屬 basic block 在 code cache 的開頭。我們把這段 host binary 覆寫了! 當然寫的內容和被覆寫的內容一模一樣。我們只想要透過這個方式反推觸發例外的 guest pc。

static inline int tcg\_gen\_code\_common(TCGContext \*s, uint8\_t \*gen\_code\_buf,
                                      long search\_pc)
{
    for(;;) {
        switch(opc) {
        case INDEX\_op\_nopn:
            args += args\[0\];
            goto next;
        case INDEX\_op\_call:
            dead\_args \= s\-&gt;op\_dead\_args\[op\_index\];
            args += tcg\_reg\_alloc\_call(s, def, opc, args, dead\_args);
            goto next;
        }
        args += def\-&gt;nb\_args;
    next:
        // 如果 offset (search\_pc) 落在 tc\_ptri (gen\_code\_buf) 和 code cache
        // 目前存放 host binary 的位址之間， 返回 TCG gen\_opc\_buf index。
        if (search\_pc &gt;= 0 &amp;&amp; search\_pc <s\>code\_ptr \- gen\_code\_buf) {
            return op\_index;
        }
        op\_index++;
    }
}

此時，gen\_opc\_pc 和 gen\_opc\_cc\_op 已存放發生例外的 guest pc 和當時的 condition code。gen\_pc\_load 負責回復 CPUState。

void gen\_pc\_load(CPUState \*env, TranslationBlock \*tb,
                unsigned long searched\_pc, int pc\_pos, void \*puc)
{
    env\-&gt;eip \= gen\_opc\_pc\[pc\_pos\] \- tb\-&gt;cs\_base;
    cc\_op \= gen\_opc\_cc\_op\[pc\_pos\];
}

至此，CPUState 已完全回復，我們回來看 tlb\_fill。raise\_exception\_err (target-i386/op\_helper.c) 這時候拉起虛擬 CPU 的 exception\_index (env->exception\_index)，並設置 error\_code (env->error\_code)。

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

raise\_exception\_err 實際上是 raise\_interrupt 的包裝 (wrapper)。QEMU\_NORETURN 前綴代表此函式不會返回。它其實是 GCC 擴展 \_\_attribute\_\_ ((\_\_noreturn\_\_))，定義在 qemu-common.h \[1\]。

static void QEMU\_NORETURN raise\_interrupt(int intno, int is\_int, int error\_code,
                                          int next\_eip\_addend)
{
    ... 略 ...
 
    env\-&gt;exception\_index \= intno;
    env\-&gt;error\_code \= error\_code;
    env\-&gt;exception\_is\_int \= is\_int;
    env\-&gt;exception\_next\_eip \= env\-&gt;eip + next\_eip\_addend;
    cpu\_loop\_exit();
}

cpu\_loop\_exit (cpu-exec.c) 用 longjmp 返回至 cpu\_exec (cpu-exec.c) 中處理例外的分支。

void cpu\_loop\_exit(void)
{
    env\-&gt;current\_tb \= NULL;
    longjmp(env\-&gt;jmp\_env, 1);
}

來看 cpu\_exec。cpu\_exec 裡用到許多 #ifdef，強烈建議查看經過預處理之後結果，即 ${BUILD}/i386-softmmu/cpu-exec.i 中的 cpu\_x86\_exec。

int cpu\_exec(CPUState \*env)
{
    // 進行翻譯並執行的迴圈。
    /\* prepare setjmp context for exception handling \*/
    for(;;) {
        if (setjmp(env\-&gt;jmp\_env) \== 0) { // 正常流程。
            /\* if an exception is pending, we execute it here \*/
            if (env\-&gt;exception\_index &gt;= 0) {
 
              /\* 2. 再來到這裡，處理例外。 \*/
 
            }
 
            next\_tb \= 0; /\* force lookup of first TB \*/
            for(;;) {
 
            } /\* inner for(;;) \*/
        }
 
        /\* 1. 我們先來到這裡。 \*/
 
    } /\* outer for(;;) \*/
}

O.K.，到這裡就是一個循環。:-) 接著，我們來驗證一下我們對 QEMU 的理解。

\[1\] http://gcc.gnu.org/onlinedocs/gcc/Function-Attributes.html
