转换时间： 2021-01-31

# QEMU Internal – Precise Exception Handling 2/5
Posted on 2012/03/15 by chenwj

Copyright (c) 2012 陳韋任 (Chen Wei-Ren)

這裡以 linux-0.11 當範例，請至 `[1][2]` 下載代碼和硬盤映像編譯。我們觀察以 `guest pc 0xe4c0` 為開頭的 basic block。使用 QEMU 0.13 [3] 運行 linux-0.11。
```
$ mkdir build; cd build
$ ../qemu-0.13.0/configure --prefix=$INSTALL --target-list=i386-softmmu \
    --enable-debug --extra-cflags="--save-temps"
$ make install
$ gdb qemu
(gdb) r -boot a -fda Image -hda hdc-0.11-new.img -vnc 0.0.0.0:1 -d in_asm,op,out_asm
```
登入後，下 `ls`。觀察 qemu.log 並定位至 0xe4c0。首先，我們可以看到如下內容:
```
IN:
0x0000e4c0:  sub    $0x4,%esp
0x0000e4c3:  mov    0x8(%esp),%eax
0x0000e4c7:  mov    %al,(%esp)
0x0000e4ca:  movzbl (%esp),%eax
0x0000e4ce:  mov    0xc(%esp),%edx
0x0000e4d2:  mov    %al,%fs:(%edx)
0x0000e4d5:  add    $0x4,%esp
0x0000e4d8:  ret

OP:
 ---- 0xe4c0
 movi_i32 tmp1,$0x4
 mov_i32 tmp0,esp
 sub_i32 tmp0,tmp0,tmp1
 mov_i32 esp,tmp0
 mov_i32 cc_src,tmp1
 mov_i32 cc_dst,tmp0

 ... 略 ...

OUT: [size=450]
0x40bbeff0:  mov    0x10(%r14),%ebp
0x40bbeff4:  sub    $0x4,%ebp

 ... 略 ...

0x4011813b:  callq  0x54d38a
0x40118140:  mov    0x10(%r14),%ebp

 ... 略 ...
```
這是 QEMU 第一次遇到尚未翻譯，以 guest pc 0xe4c0 開頭的 basic block 時所產生的輸出，這包括 guest binary (IN: 以下內容)、TCG IR (OP: 以下內容) 和 host binary (OUT: 以下內容)。

再繼續往下搜尋 0xe4c0，會看到以下內容:
```

IN:
0x0000e4c0:  sub    $0x4,%esp
0x0000e4c3:  mov    0x8(%esp),%eax
0x0000e4c7:  mov    %al,(%esp)
0x0000e4ca:  movzbl (%esp),%eax
0x0000e4ce:  mov    0xc(%esp),%edx
0x0000e4d2:  mov    %al,%fs:(%edx)
0x0000e4d5:  add    $0x4,%esp
0x0000e4d8:  ret

OP:
 ---- 0xe4c0
 movi_i32 tmp1,$0x4
 mov_i32 tmp0,esp
 sub_i32 tmp0,tmp0,tmp1
 mov_i32 esp,tmp0
 mov_i32 cc_src,tmp1
 mov_i32 cc_dst,tmp0

 ... 略 ...

RESTORE:
0x0000: 0000e4c0
0x0007: 0000e4c3
0x000d: 0000e4c7
0x0011: 0000e4ca
0x0015: 0000e4ce
0x001b: 0000e4d2
spc=0x4011813f pc_pos=0x1b eip=0000e4d2 cs_base=0
```
這裡就是重點了。spc 指的是發生例外的 host pc，eip 指的是與其相對映發生例外的 guest pc。這邊請注意，由於我們將 guest binary 翻譯成 host binary 並執行，真正發生例外的是 host binary (位於 code cache)，但是我們必須將它映射回 guest pc，查出哪一條 guest 指令發生例外，並做後續處理。我們看一下第一次翻譯 0xe4d2 所得的 host binary。
```
0x4011813b:  callq  0x54d38a
0x40118140:  mov    0x10(%r14),%ebp
```
我們可以看到 spc 0x4011813f == 0×40118140 – 1，也就是 callq 0x54d38a 下一條指令所在位址減去 1。這裡做點弊，我們在 gdb 下 print `__stb_mmu` 。
```
(gdb) print __stb_mmu
$1 = {void (target_ulong, uint8_t, int)} 0x54d38a
```
可以得知，我們在呼叫 `__stb_mmu` 的時候發生例外。`__{ld,st}{b,w,l,q}_{cmmu,mmu}` 是用來存取 guest 內存的 helper function。它們首先會查找 TLB (env1->tlb_table) 試圖取得 guest virtual address 相對映的 host virtual address。如果 TLB 命中，可直接利用該 host virtual address 存取 guest 內存內容。如果 TLB 不命中，則會呼叫 tlb_fill (target-i386/op_helper.c)。tlb_fill 會呼叫 cpu_x86_handle_mmu_fault 查找客戶頁表。如果命中，代表該 guest virtual address 所在的頁已存在，tlb_fill 會將該頁項目填入 TLB 以便後續查找。如果不命中，代表發生頁缺失，QEMU 會回復 guest CPUState，並拉起 guest exception index (env->exception_index) 通知 guest 頁缺失發生。最後交由 guest 頁缺失 handler 將該頁載入。

我先給出一個 precise exception handling 的大致流程，之後再透過閱讀代碼有更深的體會。底下給出關鍵的資料結構 TranslationBlock，它負責掌管 guest binary 和 host binary 的關係，其中 pc 代表此 basic block 起始的 guest pc，tc_ptr 指向翻譯好的 host binary 在 code cache 中的位址。
```
                                                                     code cache
          guest binary                   TranslationBlock           (host binary)

0x0000e4c0:  sub    $0x4,%esp         0x40bbeff0:  mov 0x10(%r14),%ebp
0x0000e4ca:  movzbl (%esp),%eax                                 0x40bbeff4:  sub $0x4,%ebp
0x0000e4ce:  mov    0xc(%esp),%edx
0x0000e4d2:  mov    %al,%fs:(%edx)                                      ... 略 ...
0x0000e4d5:  add    $0x4,%esp
0x0000e4d8:  ret                                                0x4011813b: callq  0x54d38a
                                               (3) offset  -->  0x40118140: mov    0x10(%r14),%ebp (1)

                                                                        ... 略 ...
```
當 QEMU 發現例外是發生在 code cache 裡，這代表需要處理 precise exception。首先，QEMU 會透過 host pc (0x4011813f) 查出相對應的 TranslationBlock – (1)。接下來，QEMU 會回復 guest CPUState。主要概念是這樣，透過之前查找到的 TranslationBlock 的 pc，我們從該 pc 所指的 guest binary 重新再翻譯一次，同時產生額外的資訊以便回復 guest CPUState – (2)。那要如何得知我們已經翻譯到發生 exception 的 guest binary? 這裡的重點在於，我们重新翻译 guest binary，直到 host binary 地址到达了出现异常的位置，这个时候 guest pc 就是产生异常的指令。我們便可以回復 guest CPUState – (3)。

接下來各小節的主題分別是:

* Precise Exception Handling 3/5 – (1)

* Precise Exception Handling 4/5 – (2) (3)

* Precise Exception Handling 5/5 – 驗證我們對 QEMU 的理解。

接下來，我們來看代碼。;)

[1] http://www.oldlinux.org/oldlinux/viewthread.php?tid=13681&extra=page%3D1

[2] http://oldlinux.org/Linux.old/bochs/linux-0.11-devel-060625.zip

[3] http://wiki.qemu.org/download/qemu-0.13.0.tar.gz
