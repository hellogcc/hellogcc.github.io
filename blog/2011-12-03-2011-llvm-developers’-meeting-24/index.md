---
title: "2011 LLVM Developers’ Meeting 2/4"
date: "2011-12-03"
categories: 
  - "llvm"
---

Copyright (c) 2011 陳韋任 (Chen Wei-Ren)

[![DSC01440](http://lh3.ggpht.com/-gufyiY1XFwk/Ts_TwMFjh1I/AAAAAAAABsk/7J4EakG9cU4/DSC01440_thumb%25255B1%25255D.jpg?imgmax=800)](http://lh4.ggpht.com/-rsLpLsfx5y8/Ts_Tvp6mraI/AAAAAAAABsg/HLL0totSLQE/s1600-h/DSC01440%25255B4%25255D.jpg)  
奇妙的感覺，我置身在 LLVM Developers’ Meeting，不再是看著事後的錄影檔。台上是 [Chris Lattner](http://nondot.org/sabre/) 給的開場致詞，說明近年來 LLVM 和 Clang 開發的情況。當然，數字不是那麼完美，所以大家就莞爾一笑帶過。我比較早進到這個會場，那時候 Chris 在台上準備。我以為他是打雜的，但是又覺得他很面熟。這時候才想起來他是 Chris Lattner。今年有三個 parallel session，有人提議明年改成兩天的議程，降成兩個 parallel session 免得有遺珠之憾。  
[![DSC01444](http://lh4.ggpht.com/-MuQhSXbt1bE/Ts_VFFQI0xI/AAAAAAAABs0/j7v0vpVEBcM/DSC01444_thumb%25255B1%25255D.jpg?imgmax=800)](http://lh5.ggpht.com/-ZwC3kvPyj4M/Ts_VEsgDl0I/AAAAAAAABsw/SEdBqyybMn8/s1600-h/DSC01444%25255B4%25255D.jpg)

第一場我選 [Extending Clang](http://llvm.org/devmtg/2011-11/#talk4)。這一場是在講如何 hook Clang 把自己想要做的功能加進去，講者是以改善編譯時期的錯誤輸出訊息當作例子。  
[![DSC01446](http://lh6.ggpht.com/-xbpDNyPtzoY/Ts_Wcl0o8DI/AAAAAAAABtE/dr8Z-S7eA5U/DSC01446_thumb%25255B1%25255D.jpg?imgmax=800)](http://lh3.ggpht.com/-3pk4wFb17H4/Ts_WcNEfB8I/AAAAAAAABtA/TQtdNTzfEx8/s1600-h/DSC01446%25255B4%25255D.jpg)

第二場我選 [PTX Back-End: GPU Programming With LLVM](http://llvm.org/devmtg/2011-11/#talk11)。我有送幾個小 patch 給講者，當然要過來捧一下場。這一場主要是講 PTX 後端目前的情況，以及針對 PTX 這種 virtual ISA 實作後端有怎樣的問題，最後以和 Nvidia CUDA 編譯器的效能比較作結。PTX 有無限多個暫存器，在實作上講者提出了幾個方法。最後選擇輸出 LLVM virtual register，最後交由 Nvidia PTX 組譯器做 register allocation。我前方這一位是 [Tanya Lattner](http://nondot.org/tonic/Welcome.html)，Chris Lattner 的老婆，挺著個大肚子。她在前座當 session 的 moderator，負責提醒講者時間限制，這是志願職。  
[![DSC01447](http://lh6.ggpht.com/-0qxErgF9BwQ/Ts_YMA0CUcI/AAAAAAAABtU/2MucHyCvktY/DSC01447_thumb%25255B1%25255D.jpg?imgmax=800)](http://lh4.ggpht.com/-2B6Y8ZGRHtk/Ts_YLnVqEqI/AAAAAAAABtQ/n45Xv9tbXkA/s1600-h/DSC01447%25255B4%25255D.jpg)  
第三場我選 [Integrating LLVM into FreeBSD](http://llvm.org/devmtg/2011-11/#talk1)。講者提到要將 GNU 工具鏈完全從 FreeBSD 剔除需要哪些工作，以及目前的進度。估計最快在 FreeBSD 10 能夠用 Clang 當作預設系統編譯器。  
[![DSC01448](http://lh5.ggpht.com/-YuJnbbASX7I/Ts_Y7ib3jqI/AAAAAAAABtk/m8oQDxRhrm4/DSC01448_thumb%25255B1%25255D.jpg?imgmax=800)](http://lh3.ggpht.com/-JPdbFd7dsGM/Ts_Y7DeLR1I/AAAAAAAABtg/iyZY448NHrg/s1600-h/DSC01448%25255B4%25255D.jpg)  
第四場我選 [LLVM MC In Practice](http://llvm.org/devmtg/2011-11/#talk8)。台上較年輕的是 Owen Anderson，台下較為年長的是 Jim Grosbach。他們分別有自己的部分要講。這部分講到將組譯器 (assembler) 整合進 LLVM 有什麼好處。之前是依賴 GNU 的組譯器，gas。JIT 架構也要改寫，改成 MCJIT。這裡主要是將以前各個元件共通但又自己有一份的部分整合在一起，這樣對維護和除錯比較好。

以上是早上的場次，接下來是午飯時間。
