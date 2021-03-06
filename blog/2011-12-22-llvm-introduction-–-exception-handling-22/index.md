---
title: "LLVM Introduction – Exception Handling 2/2"
date: "2011-12-22"
categories: 
  - "llvm"
---

Copyright (c) 2011 陳韋任 (Chen Wei-Ren)

我們繼續接著看 LLVM 針對 nested try、exception specification 和 destructor 的處理。這邊需要對 C++ 的語法和例外處理有點認識，請見 \[1\]\[2\]。首先來看 nested try。

C++ nested try 的例子如下 (請見 \[3\] 第 29 頁):

try {
    ...
    try {
        ...
        MayThrowSomething();
        ...
    } catch (int i) {
        DoSomethingWithInt(i);
    } catch (class A a) {
        DoSomethingWithA(a);
    }
    ...
} catch (class B b) {
    DoSomethingWithB(b);
} catch (...) {
    DoSomethingElse();
}

會丟出例外的 MayThrowSomething 函式其 landingpad 區塊的 LLVM IR 如下:

lpad:
  %info = landingpad { i8\*, i32 }
    personality @\_\_gxx\_personality\_v0
    catch @\_ZTIi    // catch (int)
    catch @\_ZTI1A   // catch (class A)
    catch @\_ZTI1B   // catch (class B)
    catch null      // catch (...)

這裡我們可以看到 landingpad 會列出所有可能會接住 MayThrowSomething 例外的 catch clause。

C++ 允許程序員規範一個函式所能丟出例外其型別 (exception specification)，例如 (\[3\] 第 30 頁):

// foo 不允許丟出任何例外
int foo() throw () {
    bar();
    return 0;
}

LLVM 中用 filter 來實現 exception specification，其對映的 LLVM IR 如下:

  invoke void @\_Z3barv() to label %cont unwind label %lpad

cont:
  ret i32 0

lpad:
  %info = landingpad { i8\*, i32 }
          personality @\_\_gxx\_personality\_v0
          filter \[0 x i8\*\] zeroinitializer
  %except = extractvalue { i8\*, i32 } %info, 0
  tail call void @\_\_cxa\_call\_unexpected(%except)
  unreachable

我們可以看到針對 bar 的呼叫是用 invoke，

  invoke void @\_Z3barv() to label %cont unwind label %lpad

這裡的重點在 %lpad 區塊。landingpad 後面有一個 filter，

  filter \[0 x i8\*\] zeroinitializer

它後面是一個 typeinfo 的數組 (array) \[4\]。如果丟出的例外不匹配該數組中的任一個 typeinfo，landingpad 會傳回負值。這種情況下，\_\_cxa\_call\_unexpected 或是 \_Unwind\_Resume 應該被呼叫 \[5\]。這裡我們可以看到，針對不會丟出例外的函式，filter 帶的數組是 \[0 x i8\*\]，這是一個沒有元素的數組。

  tail call void @\_\_cxa\_call\_unexpected(%except)

call 指令之前 tail 標記代表此處的函式呼叫可以做 tail call optimization 或是 sibling call optimization。詳細請見 \[6\]。其後的 unreachable 是告知優化器執行流程  
不會到達此處。以此例而言，\_\_cxa\_call\_unexpected 具有 noreturn 的屬性，因此其後會加上 unreachable，代表 \_\_cxa\_call\_unexpected 函式呼叫不會返回。

最後，我們來看 LLVM IR 如何處理 destructor。在做 stack unwinding 的時候，當前 stack 上的物件 (object) 的解構子 (destructor) 會被執行，這是 C++ 的情況。其它 語言或是語言擴展也有提供類似機制。這邊以 GCC cleanup attribute 為例 \[7\] (\[3\] 第 31 頁):

void oof(void \*);
void bar(void) {
    int x \_\_attribute\_\_((cleanup(oof)));
    foo();
    ...
}

當變數 x 離開目前的範圍，oof 會被呼叫。有兩種情況會導致變數 x 離開目前的範圍，

- bar 函式正常執行完畢。
- foo 函式丟出例外，或是其它會導致 stack unwinding 的事情發生。

我們只考慮後者，並看 LLVM 怎麼實現 destructor/cleanup，其對映的 LLVM IR 如下:

define void @bar() {
entry:
  %x = alloca i32
  invoke void @foo() to label %cont unwind label %lpad

cont:
  ...

lpad:
  %info = landingpad { i8\*, i32 }
          personality @\_\_gcc\_personality\_v0
          cleanup
  %var\_ptr = bitcast i32\* %x to i8\*
  call void @oof(i8\* %var\_ptr)
  resume { i8\*, i32 } %info
}

landingpad 後面帶的 cleanup 指明當前的區塊 (%lpad) 有 cleanup 需要執行，這裡是指 oof 函式。因為當前函式沒有合適的 catch clause 處理該例外，最後執行 resume。這裡要注意到，如果 foo 正常執行，走到 %cont 區塊，oof 最終還是會被執行。上述 LLVM IR 只看 foo 會丟出例外的情況。

眼尖的人會看到 personality routine 改叫 \_\_gcc\_personality\_v0，請見 \[9\]。

LLVM 代碼中有展示例外處理的範例，有興趣的人可以參考 examples/ExceptionDemo/ExceptionDemo.cpp，並遵照底下流程編譯，最後執行檔放在 $INSTALL 目錄。

$ wget http://llvm.org/releases/3.0/llvm-3.0.tar.gz; tar xvf llvm-3.0.tar.gz
$ mkdir build; cd build
$ ../llvm-3.0.src/configure \--prefix\=$INSTALL
$ make install
$ cd example
$ make install

限於個人才疏學淺，有興趣的人可以參考底下文件。

- http://www.llvm.org/docs/ExceptionHandling.html
- http://blog.llvm.org/2011/11/llvm-30-exception-handling-redesign.html
- http://stackoverflow.com/questions/329059/what-is-gxx-personality-v0-for
- http://www.airs.com/blog/archives/464
- http://lists.cs.uiuc.edu/pipermail/llvmdev/2011-July/041748.html
- http://lists.cs.uiuc.edu/pipermail/llvmdev/2012-January/046700.html

erlv 整理了一下 LLVM 3.0 的新特色，有興趣的人請見 [](http://www.lingcc.com/2011/12/08/11882/)你好，LLVM 3.0。

\[1\] http://www.cplusplus.com/doc/tutorial/exceptions/  
\[2\] http://en.wikibooks.org/wiki/C++\_Programming/Exception\_Handling  
\[3\] http://llvm.org/devmtg/2011-09-16/EuroLLVM2011-ExceptionHandling.pdf  
\[4\] http://llvm.org/docs/LangRef.html#t\_array  
\[5\] http://libcxxabi.llvm.org/spec.html  
\[6\] http://llvm.org/releases/3.0/docs/LangRef.html#i\_call  
\[7\] http://gcc.gnu.org/onlinedocs/gcc/Variable-Attributes.html  
\[8\] http://llvm.org/releases/3.0/docs/LangRef.html#i\_invoke  
\[9\] http://people.cs.nctu.edu.tw/~chenwj/log/GCC/richi-2011-12-16.txt
