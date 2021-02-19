转换时间： 2021-01-31

FIXME: photo

# 2011 LLVM Developers’ Meeting 3/4
Posted on 2011/12/04 by chenwj

Copyright (c) 2011 陳韋任 (Chen Wei-Ren)

DSC01449
午餐時間。聖荷西是中午，我不知道台灣時間是幾時了。總之，吃飯就對了。由左至右依序是: 洋芋片、醬料、綠色捲心麵、巧克力薄荷糖、礦泉水和三明治。

午飯過後，我四處逛了一下。首先是隔壁桌兩位上海 ARM 公司的工程師和一位當地 AMD 的中國女性工程師，我和她在早上第二場見過面。話說，東方臉孔一開口就先講英語，這有點不對頭。下次先問問對方會不會說中文。會說中文但又全程講英語怪怪的。

Google 的廖世偉 (Shih-wei Liao) 和 MTK 的唐文力 (Luba Tang) 在這邊有個 MC Linkers BOF 要主持。我趁這個機會去拜訪一下，但是只有遇到廖世偉老師和他從台大帶來的學生，我跟他們聊了一下就離開了。之後有遇到在當地 Nvidia 做編譯器的中國工程師。看了一下時間，MC Linkers BOF 已經開始了，我趕緊過去捧個場。
DSC01450
台上是穿橘色衣服的是廖世偉老師，站在門邊穿黑衣服的是唐文力。主要的主持人是唐文力。BOF (Birds of a Feather) 在這裡指的是非式的討論，主要是希望大家一起腦力激盪，提出各種可能方案並討論其優劣。最好的結果是能討論出一個共識，以免分散大家的精力。

抱歉，接下來的 Android Renderscript 有點恍神，沒照到像。這場因為前面 MC Linkers BOF 的 delay，我只能聽到後面結尾的部分。我想它的 abstract 講得算是頗明白了。下一場我選 Register Allocation in LLVM 3.0。在這兩場的中間有海報展示，並有提供下午茶點。可能是因為時差的關係，我真的很想睡，喝了多少咖啡都沒用。Register Allocation in LLVM 3.0 這一場並未提及 LLVM 3.0 新版 greedy register allocator 的運作方式。講者主要是說 spilling 在他的觀點來看比較重要，我想是相較於 register allocation。另外也提到 register allocation 不僅僅只做 register allocation，還有其它的優化也可一併做掉，印像中主要跟 spilling 有關。我想說這一場的講者其低沉嗓音真的讓我很想睡，我真的撐不住。
DSC01454
Dragonegg 的作者 Duncan Sands，因為我也送過小 patch 給他，所以過來捧個場。他在 IRC 上的代號是 baldrick，我常在 IRC 上跟他聊天。他在 ML (Mailing List) 上也很活躍。這一場是 Super-optimizing LLVM IR。主要是想自動找出某些 LLVM 應該優化卻沒有優化的代碼片段，之後再以人工實現那些優化。這邊他提到 AST 上的 type 和 value 應該設定成最小值，比如說 i1 和 i4 應該選 i1 作為 AST 節點上的型態，之後進行查找，看能不能找出應該優化卻沒有優化的代碼片段。最後他給出優化前後的效能比較。很可惜，並非每一隻 benchmark 都有效能上的增長。即使是最應該拿到 performance gain 的 const propagation 都有例外。他的解釋是做完 const propagation 之後，branch 的情況有變，進而影響效能。這部分的分析我不很確定，必須要等錄影和投影片上傳到 2011 LLVM Developers’ Meeting 之後才能確定。

最後我選 Backend/Infrastructure Super BoF 做為今天最後一場議程。這一場主要是 AMD 的工程師和 PTX 後端的作者提出他們在實作上的問題，似乎是在尋求比較正式的解法，或者是說 LLVM Infrastructure 是否有哪些地方需要調整以滿足他們的需求。在這一場議程一開始的時候，我找了個 Justin Holewinski (PTX 後端目前主要的作者) 附近的座位坐下，然後很冒失的遞上我的名片。我很高興他認得我。剛巧 Dan Bailey (PTX 後端目前主要的作者) 也坐在 Justin Holewinski 旁邊，我趕快再遞上一張，他也認得我。

在離開會場前去晚宴之前，我想找 Duncan Sands 見個面。一樣，我也很冒失的遞上我的名片。一開始，他還不清楚我是誰。後來他看了一下我的電子郵件帳號，問我是不是在 IRC 上的跟他聊天的那一個。BINGO! 他從他皮包裡拿出一張名片 (可能是僅有的一張) 給我，並說他很少用這個。在一起聊天的還有 Nadav Rotem，Intel OpenCL SDK Vectorizer 的作者，當然還有其他人，不過我只認得他。這時候大家開始掏名片了，趁機再遞上一張名片給 Nadav Rotem。

這時候聊的比較非技術性。Duncan、Nadav 和其他人討論父母親那一輩所生子女個數會不會影響到他們這一輩所生子女個數。Duncan 說他有這樣的一個假說說是會有影響，他老婆那一邊兄弟姊妹不計其數，而他這邊兄弟姊妹不多，所以他跟他老婆就只有生四個，算是取中間值。印象中，Nadav 要不是獨生子就是沒有很多兄弟姊妹，他的小孩也不多。Duncan 轉過頭來問我說你有幾個小孩。我回答他說我還沒結婚。他接著問我兄弟姊妹多少，我說中間。他再問我我打算生幾個，我還是說中間 (我想生三個)。ㄜ…，算是贊成 Duncan 的假說了。Duncan 這邊的聊天散場之後，我去找 MTK 那一組人，我打算晚宴就找會講中文的一起吃，省的尷尬。
