---
title: "LLVM Introduction – How to use LLVM JIT 2/3"
date: "2011-10-25"
categories: 
  - "llvm"
---

Copyright (c) 2011 陳韋任 (Chen Wei-Ren)

2\. 實驗 1/2 – 建立 LLVM Module

我們來動手一試吧! 本實驗參考[LLVM Tutorial 2: A More Complicated Function](http://llvm.org/releases/2.6/docs/tutorial/JITTutorial2.html)，但使用的是 LLVM 2.9 Release。注意! LLVM API 較不穩定，請根據 LLVM 實際版本參照相對應的文件。

本實驗是要用 LLVM IR 實現一個 Greatest Common Divisor (GCD) 演算法，最後使用 LLVM JIT 生成 binary 並執行。GCD 演算法的 C 代碼如下:

unsigned gcd(unsigned x, unsigned y) {
 
    if ( x \== y ) {
        return x;
    } else if ( x < y ) {
        return gcd( x, y \- x );
    } else {
        return gcd( x \- y, y);
    }
 
}

上述代碼的 control flow graph (CFG) 請見 [LLVM Tutorial 2: A More Complicated Function](http://llvm.org/releases/2.6/docs/tutorial/JITTutorial2.html)。好! 我們開始吧。:-) 使用 LLVM JIT 基本上有底下幾個步驟:

- 創建或是讀入 LLVM Module。Module 是 LLVM IR 的容器。先在 Module 中建立函式，再於函式中建立基本塊，最後在基本塊中填入 LLVM IR。\[1\]\[2\]\[3\]
- 於我們所建立的 LLVM IR 上施行優化 (可選)。在 LLVM 中，優化的過程稱為 pass。pass 也可以是僅將 LLVM IR 做些轉換而非優化。\[4\]
- 呼叫 LLVM JIT，將 LLVM IR 編譯成 host binary。
- 取得被編譯成 host binary 的 function pointer。透過呼叫該 function pointer 執行並取得結果。

本實驗做到步驟 2，下一個實驗會補上 3 和 4。因為 LLVM API 較不穩定，請善用 [LLVM API Documentation](http://llvm.org/doxygen/) 查詢 API 使用方法。

/\*
 \*  $ g++ tut2.cpp \`llvm-config --cxxflags --ldflags --libs all\` -o tut2
 \*  $ ./tut2
 \*/
 
#include "llvm/LLVMContext.h"
#include "llvm/Module.h"
#include "llvm/Function.h"
#include "llvm/PassManager.h"
#include "llvm/Analysis/Verifier.h"
#include "llvm/Assembly/PrintModulePass.h"
#include "llvm/Support/FormattedStream.h"
#include "llvm/Support/IRBuilder.h"
 
using namespace llvm;
 
Module\* makeLLVMModule(LLVMContext& ctx); // 負責創建 LLVM Module。
 
int main(int argc, char\*\*argv) {
 
  // LLVMContext 是較晚期才加入 LLVM 的類別。
  // 其作用是管理 LLVM core infrastructure 中的 global data。
  // 多執行緒的情況下，每個執行緒都應該要有自己的 LLVMContext。
  // 請見 llvm/LLVMContext.h。 
  LLVMContext Context;
 
  Module\* Mod \= makeLLVMModule(Context); // 呼叫 makeLLVMModule 取得 LLVM Module。
 
  verifyModule(\*Mod, PrintMessageAction); // 驗證此 Module 是否合法。
 
  PassManager PM; // 所有的轉換或是優化均被視為 pass，由 PassManager 進行調度。
 
  PM.add(createPrintModulePass(&outs())); // 加入打印 LLVM Module 內容的 pass。
  PM.run(\*Mod); // 於該 Module 運行 pass。
 
  delete Mod;
  return 0;
}
 
Module\* makeLLVMModule(LLVMContext& ctx) {
  /\* 尚未實作 \*/
}

現在我們知道大致的架構，接著我們來看怎麼實現 makeLLVMModule。這裡做點弊，我們先來看最後 LLVM IR 的輸出，先有個感覺。;-)

; ModuleID = 'tut2'

define i32 @gcd(i32 %x, i32 %y) {
entry:
  %tmp = icmp eq i32 %x, %y
  br i1 %tmp, label %return, label %cond\_false

return:                                           ; preds = %entry
  ret i32 %x

cond\_false:                                       ; preds = %entry
  %tmp2 = icmp ult i32 %x, %y
  br i1 %tmp2, label %cond\_true, label %cond\_false1

cond\_true:                                        ; preds = %cond\_false
  %tmp3 = sub i32 %y, %x
  %tmp4 = call i32 @gcd(i32 %x, i32 %tmp3)
  ret i32 %tmp4

cond\_false1:                                      ; preds = %cond\_false
  %tmp5 = sub i32 %x, %y
  %tmp6 = call i32 @gcd(i32 %tmp5, i32 %y)
  ret i32 %tmp6
}

開始囉!

Module\* makeLLVMModule(LLVMContext& Context) {
 
  Module\* M \= new Module("tut2", Context);
 
  // 在 Module 中插入新的函式 (Function)。若該函式已存在，返回該函式。
  Function \*gcd \=
      cast(M\->getOrInsertFunction("gcd",
      /\* 返回型別 \*/                        Type::getInt32Ty(Context),
      /\* 參數 \*/                            Type::getInt32Ty(Context),
      /\* 參數 \*/                            Type::getInt32Ty(Context),
      /\* 結尾 \*/                            (Type \*)0));
 
  // 分別將 gcd 的參數命名為 x 和 y。使將來的輸出較易懂。
  Function::arg\_iterator args \= gcd\->arg\_begin();
  Value\* x \= args++;
  x\->setName("x");
  Value\* y \= args++;
  y\->setName("y");
 
  // 在函式中插入基本塊 (BasicBlock)。基本塊內含 LLVM IR，並以
  // terminator instruction 結尾，請見
  // http://llvm.org/docs/LangRef.html#terminators
  //
  // 底下建立的基本塊，請參照
  // http://llvm.org/releases/2.6/docs/tutorial/JITTutorial2.html
  // 中的 CFG。
  //
  BasicBlock\* entry \= BasicBlock::Create(Context, "entry", gcd);
  BasicBlock\* ret \= BasicBlock::Create(Context, "return", gcd);
  BasicBlock\* cond\_false \= BasicBlock::Create(Context, "cond\_false", gcd);
  BasicBlock\* cond\_true \= BasicBlock::Create(Context, "cond\_true", gcd);
  // 
  // 即使我們賦予 cond\_false 和 cond\_false\_2 相同的名稱，LLVM 會將
  // 之替換成不同名稱。如此可省去我們命名的麻煩。
  //
  BasicBlock\* cond\_false\_2 \= BasicBlock::Create(Context, "cond\_false", gcd);
 
  /\* 開始填入 LLVM IR \*/
 
  // IRBuilder 提供一組一致的介面生成 LLVM IR。
  IRBuilder builder(entry); // 於基本塊 entry 填入 LLVM IR。
  //
  //  %tmp = icmp eq i32 %x, %y
  //  br i1 %tmp, label %return, label %cond\_false
  //
  Value\* xEqualsY \= builder.CreateICmpEQ(x, y, "tmp");
  builder.CreateCondBr(xEqualsY, ret, cond\_false);
  builder.SetInsertPoint(ret); // 於基本塊 ret 填入 LLVM IR。
  //
  // ret i32 %x
  //
  builder.CreateRet(x);
 
  builder.SetInsertPoint(cond\_false); // 於基本塊 cond\_false 填入 LLVM IR。
  //
  //  注意! LLVM 中的 integer type 不帶有 signed 或是 unsigned 的資訊。
  //  icmp 需要顯式的對其 integer type 運算元做 signed 或是 unsigned
  //  的解釋。請見 http://llvm.org/docs/LangRef.html#i\_icmp
  //
  //  %tmp2 = icmp ult i32 %x, %y
  //  br i1 %tmp2, label %cond\_true, label %cond\_false1
  //
  Value\* xLessThanY \= builder.CreateICmpULT(x, y, "tmp");
  builder.CreateCondBr(xLessThanY, cond\_true, cond\_false\_2);
 
  builder.SetInsertPoint(cond\_true); // 於基本塊 cond\_true 填入 LLVM IR。
  //
  //  %tmp3 = sub i32 %y, %x
  //  %tmp4 = call i32 @gcd(i32 %x, i32 %tmp3)
  //  ret i32 %tmp4
  //
  Value\* yMinusX \= builder.CreateSub(y, x, "tmp");
  std::vector args1;
  args1.push\_back(x);
  args1.push\_back(yMinusX);
  Value\* recur\_1 \= builder.CreateCall(gcd, args1.begin(), args1.end(), "tmp");
  builder.CreateRet(recur\_1);
 
  builder.SetInsertPoint(cond\_false\_2); // 於基本塊 cond\_false\_2 填入 LLVM IR。
  //
  //  %tmp5 = sub i32 %x, %y
  //  %tmp6 = call i32 @gcd(i32 %tmp5, i32 %y)
  //  ret i32 %tmp6
  //
  Value\* xMinusY \= builder.CreateSub(x, y, "tmp");
  std::vector args2;
  args2.push\_back(xMinusY);
  args2.push\_back(y);
  Value\* recur\_2 \= builder.CreateCall(gcd, args2.begin(), args2.end(), "tmp");
  builder.CreateRet(recur\_2);
 
  return M;
}

請注意! 生成 LLVM IR 時，請遵守 [LLVM Language Reference Manual](http://llvm.org/docs/LangRef.html) 上的規範。某些情況被 LLVM 視為未定義 (undefined)，這代表 LLVM 想怎麼做都可以。千萬不要憑直覺解讀運算後的結果，否則你怎麼死的都不知道。

舉例: shl 將第一個運算元往左移位指定的位數。你先想想底下的結果為何。

    shl i32 1, 32 ; 把長度為 32-bit 的 1 往左移 32 位

是 0 嗎? 我們來看規格怎麼說。:-)

Semantics:

  The value produced is ... . If op2 is (statically or dynamically) negative or
equal to or larger than the number of bits in op1, the result is undefined.

是的，shl i32 1, 32 所得結果是 undef。這代表你的程序可能能正確執行、當機或是任何事都可能發生。請嚴格遵守 [LLVM Language Reference Manual](http://llvm.org/docs/LangRef.html) 上的規範。不要自作聰明。

\[1\] http://llvm.org/docs/ProgrammersManual.html#Module  
\[2\] http://llvm.org/docs/ProgrammersManual.html#Function  
\[3\] http://llvm.org/docs/ProgrammersManual.html#BasicBlock  
\[4\] http://llvm.org/docs/WritingAnLLVMPass.html
