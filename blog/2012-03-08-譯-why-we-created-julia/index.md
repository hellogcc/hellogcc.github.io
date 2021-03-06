---
title: "[譯] Why We Created Julia"
date: "2012-03-08"
categories: 
  - "其它技术"
---

Copyright (c) 2012 陳韋任 (Chen Wei-Ren)

緣起

在 LLVM 郵件列表上看到一封介紹一款程式語言 Julia，後端基於 LLVM JIT \[1\]。Julia 的目標非常宏大，它想要擁有當今大多數語言有的優點，卻沒有其缺點。我對它的目標不做評論。在介紹作者們為何要創造 Julia 這一語言的文章中，我注意到 CSDN 有篇中文翻譯，翻得有點拗口。這類的技術文章通常都需要有一定背景知識的人才能較好的掌握原作者的意思。在此，獻上此篇中文翻譯。限於個人能力有限，歡迎不吝批評指教。

原文: [Why We Created Julia](http://julialang.org/blog/2012/02/why-we-created-julia/)

簡單來說，因為我們很貪心。

我們是重度 Matlab 使用者。其中有些人是 Lisp 黑客，有些人是 Python 愛好者，其他人是 Ruby 愛好者，甚至有人是 Perl 黑客。我們其中有些人在我們毛還沒長齊之前就在使用 [Mathematica](http://en.wikipedia.org/wiki/Mathematica)。有些人甚至是女性。我們使用 [R 語言](http://en.wikipedia.org/wiki/R_(programming_language))產生許多統計圖型。C 是我們最為喜愛的語言 \[2\]。

我們熱愛所有這些語言，它們是如此美妙且強大。就我們所工作的領域 – 科學計算，機器學習，資料探勘，大規模線性計算，分散與平行計算 – 每一種語言對某些領域來說堪稱完美，但對其它領域卻是惡夢。使用一種語言都是一種權衡的藝術。

我們很貪心，我們想要更多。

我們想要一種語言，它必須是開源的，採自由授權。我們希望 C 的速度，Ruby 的動態性。我們想要一種語言，其資料和代碼同一格式 ([homoiconic](http://en.wikipedia.org/wiki/Homoiconic))，同 Lisp 一般擁有真的巨集，但卻使用像 Matlab 那樣明顯和熟悉的數學符號來加以表示。我們想要一種像 Python 一樣對於各種領域問題都如此好用 ([general programming](http://en.wikipedia.org/wiki/General-purpose_programming_language)) 的語言。對於統計方面，像 R 一樣容易使用。對於字串處理，又像使用 Perl 一般自然。對於線性代數，像 Matlab 般強大。像 shell 一樣，可以用來膠合程序中其它的組件。易於學習，卻又能讓最為嚴肅的黑客高興。我們希望它是互動式，又同時是編譯式的語言。

(我們有提到它必須有 C 一般的速度，對吧?)

我們是苛刻的，我們想要一種語言能提供如 Hadoop 那樣強大的分散式計算 – 沒有臃腫的 Java 和 XML 代碼; 不需要被強迫去過濾放在數以百計機器上的海量日誌來找出臭蟲。我們想要語言有強大的表達能力，卻又沒有令人費解的複雜性。我們想要寫出一個簡單、做純量計算的迴圈，它可以被編譯成只使用到暫存器的機器碼。我們想要寫出 A \* B 的矩陣計算，並同時在數千台機器上發起數千個計算，計算龐大的矩陣乘積。

我們不想提到型別，當我們不喜歡它的時候。但當我們需要多型函式 (polymorphic function)，我們希望使用泛型編程 ([generic programming](http://en.wikipedia.org/wiki/Generic_programming)) 撰寫演算法，只寫一次，並將該演算法套用在無窮多的型別上。我們希望使用多分派 ([multiple dispatch](http://en.wikipedia.org/wiki/Multiple_dispatch)) 來有效的根據函式所有參數選擇一個最適當的實現，並為截然不同的型別提供共通的功能。僅管擁有如此強大的能力，我們希望這個語言能簡單且乾淨。

我們似乎要求太多了，對嗎?

即使我們認識到我們是無可救藥的貪婪，我們仍舊想要上述所有的特性。大約在兩年半以前，我們開始創造一種滿足我們貪婪的語言。它還不完備，但是該是時候發布 1.0 正式版了 – 我們所創造的語言叫做 Julia。它已經滿足我們 90% 無理的要求，現在它需要其他人無理的要求讓它更完美。所以，如果你同樣也是一個貪婪、不可理喻、苛刻的程序員，我們希望你前來一試。

\[1\] [Introducing julia, and gauging interest in a julia BOF session at the upcoming LLVM conference in London](http://lists.cs.uiuc.edu/pipermail/llvmdev/2012-March/048093.html)  
\[2\] http://people.cs.nctu.edu.tw/~chenwj/log/LLVM/jey-2012-03-08.txt
