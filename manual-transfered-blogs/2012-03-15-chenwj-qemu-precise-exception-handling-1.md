转换时间： 2021-01-31

# QEMU Internal – Precise Exception Handling 1/5
Posted on 2012/03/15 by chenwj

Copyright (c) 2012 陳韋任 (Chen Wei-Ren)

一般計算機架構會定義當例外/中斷發生時，當下的 CPU 狀態應當為何，這個稱之為 precise exception。假設當下流水線是底下這樣:
```
  A. addl %eax,(%esp)
  B. addl %ebx,(%esp)
  C. movl %esi,(%ebp)
  D. subl %ecx,5
```
當執行到指令 (C)，欲從 %ebp 所指的內存位址讀取資料至 %esi 時，發生頁缺失例外。以 x86 對 precise exception 的定義，在指令 (C) 之前的指令，即 (A) 和 (B) 其結果必須完成。也就是說，暫存器/內存的內容應該更新; 在指令 (C) 之後的指令，即 (D) 的結果必須捨棄。此範例取自 [1]。(註: 對此段匯編的解讀，我直接引用 [1]。如果單從 AT&T 語法來看，指令 (C) 應為將 %esi 其值寫至 %ebp 所指位址。)

Precise exception 在 binary translation 中佔有重要地位。在此以 QEMU 為例，說明 QEMU 如何確保 pecise exception。由以上對 pecise exception 的說明，我們可以知道 precise exception 必須考量暫存器和內存。在 binary translation 中，我們關注的是客戶 (guest) 的 precise exception。因此，我們必須確保當 guest 代碼發生例外時，guest 的暫存器和內存其內容必須滿足 precise exception 的要求。這樣 guest 的 exception handler 才能正確處理該例外。

就 guest 暫存器而言，在 QEMU 中需要考量的是 CPUState。QEMU 在每一個可能會發生例外的指令或是 helper function 之前，會將 CPUState 中的大部分內容更新，少數未更新的內容會在 guest 真正發生例外時重新再計算。以 x86 為例，pc 和 condition code 屬於後者。在 binary translation 中，出於效能上的考量，通常會在一個 basic block 的結尾才更新 guest pc，而非每翻譯一個 guest 指令就更新 guest pc。

就 guest 內存而言，我們將重點放在 guest memory store operation 上，因為只有 memory store operation 會修改 guest 內存的內容。第一，QEMU 會依照 guest 原本 memory store operation 的順序進行翻譯。第二，針對所有潛在會發生例外的 guest 指令，QEMU 保留其相對於 guest memory store operation 的順序。簡單來說，QEMU 不會 reorder guest 指令順序。這簡化了 QEMU 維護 precise exception 的複雜度，但同時也喪失了一些潛在可能的優化 [2][3]。硬體对异常的处理可能还有别的行为，只要滿足 precise exception 的規範，同样是可以接受的。在 QEMU，我们选择了一个最为保守的实现方式。

接下來以 guest 頁缺失例外為例，讓我們來觀察當 guest 發生頁缺失時，QEMU 是如何維護正確的 guest 暫存器和內存。

[1] The Technology Behind Crusoe™ Processor
[2] http://lists.gnu.org/archive/html/qemu-devel/2012-02/msg04138.html
[3] http://people.cs.nctu.edu.tw/~chenwj/log/QEMU/agraf-2012-03-02.txt
