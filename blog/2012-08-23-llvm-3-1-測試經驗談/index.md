---
title: "LLVM 3.1 測試經驗談"
date: "2012-08-23"
categories: 
  - "llvm"
---

Copyright (c) 2012 陳韋任 (Chen Wei-Ren)

\*\*\* 此篇文章技術性不高，純粹是分享替 LLVM 做測試的經驗。\*\*\*

一切都從 [\[3.1 Release\] Call For Testers!](http://lists.cs.uiuc.edu/pipermail/llvmdev/2012-March/048355.html) 這一封徵求 LLVM 3.1 Release Tester 的信開始。我純粹抱著試試看的心態，回信給 Bill Wendling 問他新手如我是否也能參與 LLVM 3.1 Release Testing。在約莫一個禮拜之後，Bill 通知所有曾經回信給他的人關於 LLVM Testing 大致的流程以及相關注意事項。我這裡簡單做個摘要:

1\. 一開始 Bill 會要求所有測試者寄給他測試平台的相關資訊，諸如作業系統和編譯器版本。Bill 需要這個資訊以便知道測試覆蓋程度如何。以我的例子，我會回覆他:

Two pandaboard:

arm linux (ubuntu 11.04), gcc 4.5.2

注意! 基本上不鼓勵在測試期間更新作業系統和編譯器版本。

2\. Bill 會在郵件列表上公布測試的時程表，一般測試為期一個月。這次公布的時程如下:

April 16th: Branching for 3.1 release April 16-22: Phase 1 testing April 23-29: Bug fixing for Phase 1 issues, all features completed April 30-May 6: Phase 2 testing May 7-13: Addressing Phase 2 issues, final binary generation May 14th: Release of 3.1!

基本上是維持一個禮拜測試，下一個禮拜修正錯誤的循環。Bill 會視情況決定是否延長測試的時間。

3\. 在測試期間所發現的臭蟲，一律送交 [Bugzilla](http://llvm.org/bugs)。Bill 會要求你回報臭蟲的同時，將他加進 CC list 以便他監控臭蟲修正的進度。基本上，只要不是太嚴重的臭蟲都不會阻礙 LLVM release 的時程。另外，非主流平台 (x86 除外) 上的臭蟲通常都需要測試者幫忙修正。測試者可以在郵件列表或是聊天室尋求幫助。

4\. ${LLVM\_SRC}/utils/release 目錄底下提供幾個腳本協助測試者進行測試。測試分為兩類: 一類是所謂的回歸測試 (regression test)，這類測試位在 ${LLVM\_SRC}/test 底下。主要用來確保 LLVM/Clang 的正確性。第二類是評量 LLVM 在效能上是否有退步，這類測試位在 ${LLVM\_SRC}/projects/test-suite。測試分底下三個步驟:

a. 以不同模式 (Debug、Assert 或是 Release) 編譯 LLVM/Clang 3.1 pre-release candidate，也就是所謂的 RC (release candidate)，並進行回歸測試。這一步最為 重要，事實上我這次的測試也只有做到這一步。

b. 下載 LLVM/Clang 3.0 代碼並編譯，運行 test-suite 得到一組分數作為比較基準 (baseline)。

c. 以編譯好的 LLVM/Clang 3.1 pre-release candidate "手動" 運行 test-suite， 並得到一組分數。將此分數與前述 baseline 做比較，若有退步便回報。

整個測試工具的使用可以參考 [LLVM Testing Infrastructure Guide](http://llvm.org/docs/TestingGuide.html)。

O.K.，現在就等 LLVM 從 trunk 替 3.1 release 開一個 branch，接著就可以準備測試。Bill 會在郵件列表上發布 [3.1 Has Branched](http://lists.cs.uiuc.edu/pipermail/llvmdev/2012-April/048961.html) 的消息。但別急! 一切都等待 Bill 這位 release manager 的指示，他會通知測試者 release candidate 的代碼該從哪裡下載，並給出較之前為詳細的指示。你可能會收到類似 "Release Candidate 1 Ready for Testing" 的信件，我這裡簡單做個摘要:

1\. 隨信的附件中會有幾個腳本供測試者使用，你同樣可以在 ${LLVM\_SRC}/utils/release 找到底下幾個腳本:

- test-release.sh
- findRegressions-simple.py
- findRegressions-nightly.py

後兩個是用來比較 test-suite 分數。我主要使用的是 test-release.sh 這支腳本。

2\. Bill 會描述如何使用 test-release.sh。以測試 LLVM 3.1 RC1 為例，

$ test-release.sh -release 3.1 -rc 1 -no-64bit -no-compare-files

因為我是測試 ARM 平台，所以不測試 64 bit。整個編譯流程跟 GCC 極為類似，總共 有三個階段。Phase 1 是以系統工具鏈編譯 LLVM/Clang，Phase 2 是使用 Phase 1 編譯所得的 clang 編譯 LLVM/Clang，Phase 3 是使用 Phase 2 編譯所得的 clang 編譯 LLVM/Clang，最後比較 Phase 2 和 Phase 3 的結果。因為某些不知名的原因， Bill 稱此次 Phase 2 與 Phase 3 的目的檔有所不同，但可忽略，他建議我們使用 -no-compare-files 跳過 Phase 2 與 Phase 3 的比較。\`test-release.sh -help\` 可以得到更多訊息。注意! 請確定最後執行回歸測試所用的是 Phase 3 的 LLVM/Clang。剛開始進行測試的時候，我多下了 "-disable-clang" 這個選項，導致 只有 Phase 1 的結果，用 Phase 1 進行回歸測試得到的結果是不準確的。回歸測試 的結果相當重要，有臭蟲皆需要立即回報給 Bugzilla (http://llvm.org/bugs)， 並 CC 給 Bill，由他決定該臭蟲是否會阻礙 LLVM release 的時程。

3\. 分別以 LLVM 3.0 和 LLVM 3.1 RC 運行 test suite，並以 findRegressions-simple.py 和 findRegressions-nightly.py 比較兩者的結果。這次我並沒有認真地運行 test suite。

4\. 打包 Phase 3 所得結果，並交由 Bill 上傳至 LLVM 官網。基本指令如下:

$ cd rc1/Phase3/Release $ tar zcvf llvmCore-3.1-rc1.install-arm-ubuntu\_11.04.tar.gz llvmCore-3.1-rc1.install/

至此就算是告一個段落，爾後的 RC2 亦或是 RC3 基本上都是依循上述流程。這次測試過程中有個小插曲，RC2 的代碼在 ARM 上編譯不過。這時要盡快回報給 Bill，同時自己這邊也要做 bisect 抓出出錯的版本號。基本流程如下:

1\. 取得正常可運作的版本號，這裡我取 RC1 (r155062)。

$ svn log http://llvm.org/svn/llvm-project/llvm/tags/RELEASE\_31/rc1 --stop-on-copy ------------------------------------------------------------------------ r155062 | void | 2012-04-19 06:10:56 +0800 (四, 19 4 2012) | 1 line Creating release candidate rc1 from release\_31 branch

2\. 取得出錯的版本號，這裡我取 RC2 (r156037)。

$ svn log http://llvm.org/svn/llvm-project/llvm/tags/RELEASE\_31/rc2 --stop-on-copy ------------------------------------------------------------------------ r156037 | void | 2012-05-03 08:11:00 +0800 (四, 03 5 2012) | 1 line Creating release candidate rc2 from release\_31 branch

3\. 透過 svn-bisect 用二分法找出一開始出問題的版本。注意! LLVM 和 Clang 分屬不同的 SVN，但是在糾錯時請保持 LLVM 和 Clang 的版本一致。

$ cd rc2/llvm.src # 好 壞 $ svn-bisect start 155062 156037 Switching to r155672 ... # 手動將 Clang 切換至 r155672 $ cd rc2/cfe.src $ svn up -r 155672 # 開始編譯。-no-checkout 是要求直接使用當前目錄下的代碼，不從 SVN checkout。 $ cd rc2 $ ./test-release.sh -release 3.1 -rc 2 -no-checkout -no-64bit -no-compare-files # 如果 r155672 編譯出錯，將其標定為 bad，svn-bisect 會切換到下一個應該要檢測的 # 版本。 $ svn-bisect bad 155672 Switching to r155374 ...

幸運的是，在我 bisect 出錯誤版本之前，Bill 已經先我一步找到出問題的版本號。 要知道在 ARM 機器上編譯 LLVM/Clang 可以頗為耗時的一件事。:-)

4\. 等待 Bill 重新發布下一個修正過後的測試版本，RC3。

最後進入 final release 時，測試者只需編譯好 LLVM/Clang，無須再跑回歸測試或是 test suite，直接將 Phase 3 打包送給 Bill 即可。剩下的就是靜待 LLVM 3.1 Release 的發布。我這裡提一下提交臭蟲至 [Bugzilla](http://llvm.org/bugs) 時，應該要有的格式。以 [Test case Sema/arm-neon-types.c fail on ARM](http://llvm.org/bugs/show_bug.cgi?id=12694) 為例，標題寫明是在 ARM 平台上，Sema/arm-neon-types.c 這個測試失敗。在內容描述裡面，把 \`make check-all\` 所吐出的訊息貼上去。基本上 test-release.sh 這支腳本在編譯完 Phase 3 LLVM/Clang 後會運行回歸測試，即 \`make check-all\`，同時會留下 log。你只需要將該 log 的內容檢視一遍，將出錯的測試回報給 [Bugzilla](http://llvm.org/bugs) 即可。非 x86 平台的測試者需要自己主動修正錯誤，你可以在郵件列表和聊天室尋求幫助。

修正臭蟲之後就可以送 patch，以 [Fix test case failure due to C++ ABI difference on ARM](http://lists.cs.uiuc.edu/pipermail/cfe-commits/Week-of-Mon-20120430/056982.html) 為例，標題簡明扼要的描述這個 patch 修正了什麼問題，信件正文再較為詳細的敘述這個 patch 的意圖，必要時附帶上輔助資料，最後以附件形式附上 patch。

整個測試過程中，如果有 "任何" 疑問，不要遲疑，發信到郵件列表或是 release manager 尋求協助。最後，你可以在 Chris Lattner 發布的 [LLVM 3.1 Release!](http://lists.cs.uiuc.edu/pipermail/llvm-announce/2012-May/000041.html) 公告上看見你的名字。

本篇文章沒有技術含量，純粹是希望鼓勵更多人參與 LLVM 的開發，不論是以何種形式。:-)

\[1\] http://lists.cs.uiuc.edu/pipermail/llvmdev/2012-March/048355.html \[2\] http://llvm.org/docs/TestingGuide.html \[3\] http://lists.cs.uiuc.edu/pipermail/llvmdev/2012-April/048961.html \[4\] http://llvm.org/bugs/show\_bug.cgi?id=12694 \[5\] http://lists.cs.uiuc.edu/pipermail/cfe-commits/Week-of-Mon-20120430/056982.html \[6\] http://lists.cs.uiuc.edu/pipermail/llvm-announce/2012-May/000041.html
