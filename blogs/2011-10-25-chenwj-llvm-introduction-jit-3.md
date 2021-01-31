原始博客链接：

转换时间：
2021-01-31

注意：由于2011年HelloGCC博客遭遇了一次VPS掉电丢失，所有2011年之前（含）的博客都是从 Google Reader 等RSS中恢复的。损失了大部分的内容，非常的遗憾。如果你的电脑中保存过HelloGCC2012年之前的报告，欢迎发送给我们，非常感谢。请转发给 hellogcc.workgroup@gmail.com

# LLVM Introduction – How to use LLVM JIT 3/3
Posted on 2011/10/25 by chenwj

Copyright (c) 2011 陳韋任 (Chen Wei-Ren)

2. 實驗 2/2 – 使用 LLVM JIT

ExecutionEngine 是一個關鍵的類別。它能把 Module 中的特定 Function 動態編譯成 host binary。此外，它還提供介面供使用者提取 JIT-ted function (host binary) 的資訊。

本實驗補上前篇實驗的後半部。附件是完整的代碼。
```
#include "llvm/LLVMContext.h"
#include "llvm/Module.h"
#include "llvm/Function.h"
#include "llvm/PassManager.h"
#include "llvm/Analysis/Verifier.h"
#include "llvm/Assembly/PrintModulePass.h"
#include "llvm/Support/FormattedStream.h"
#include "llvm/Support/IRBuilder.h"

// LLVM JIT 所需的頭文件
#include "llvm/ExecutionEngine/JIT.h"
#include "llvm/ExecutionEngine/GenericValue.h"
#include "llvm/Target/TargetSelect.h"
#include "llvm/Support/ManagedStatic.h"

using namespace llvm;

Module* makeLLVMModule(LLVMContext& ctx);

int main(int argc, char**argv) {

  // 根據 host 初始化 LLVM target，即針對哪一個 target 生成 binary。
  InitializeNativeTarget();

  LLVMContext Context;

  Module* Mod = makeLLVMModule(Context);

  verifyModule(*Mod, PrintMessageAction);

  PassManager PM;

  PM.add(createPrintModulePass(&outs()));
  PM.run(*Mod);

  // 根據給定的 Module 生成 ExecutionEngine。
  ExecutionEngine* EE = EngineBuilder(Mod).create();

  // 在 Module 中插入新的函式 (Function)。若該函式已存在，返回該函式。
  // 因此這邊取回 makeLLVMModule 所插入的 gcd 函式。
  Function *GCD =
      cast(Mod->getOrInsertFunction("gcd",
      /* 返回型別 */                        Type::getInt32Ty(Context),
      /* 參數 */                            Type::getInt32Ty(Context),
      /* 參數 */                            Type::getInt32Ty(Context),
      /* 結尾 */                            (Type *)0));

  // 準備傳給 GCD 的參數。
  std::vector Args(2);
  Args[0].IntVal = APInt(32, 17);
  Args[1].IntVal = APInt(32, 4);

  // 將參數傳給 gcd。將其 JIT 成 host binary 並執行。取得執行後的結果。
  GenericValue GV = EE->runFunction(GCD, Args);

  outs() << "---------\nstarting GCD(17, 4) with JIT...\n";
  outs() << "Result: " << GV.IntVal <freeMachineCodeForFunction(GCD);
  delete EE;
  llvm_shutdown(); // 釋放用於 LLVM 內部的資料結構。
  return 0;
}
```
眼尖的你應該會發現我似乎忘了 delete Mod。很好! 但是你一但加上 delete Mod，執行之後會出現 segmentation fault。試試看! 😉

答案在 EngineBuilder 的註解裡 “the created engine takes ownership of the module”。
```
  /// EngineBuilder - Constructor for EngineBuilder.  If create() is called and
  /// is successful, the created engine takes ownership of the module.
  EngineBuilder(Module *m) : M(m) {
    InitEngine();
  }
```
我還漏了底下這一行沒講清楚。
```
  verifyModule(*Mod, PrintMessageAction);
```
什麼時候 Module 是非法的? 舉例來說，假設 Module 裡面有底下這樣一條 LLVM 指令。
```
  %x = add i32 1, %x ; x = 1 + x
```
直覺上來看很正常。但是 LLVM IR 滿足 SSA (Static Single Assignment) 的形式，亦即每條賦值指令都會生成一個新的變數。所以上面這條指令應該是底下這樣:
```
  %x1 = add i32 1, %x ; x1 = 1 + x
```
SSA 能夠簡化資料流 (data flow) 的分析，有助於後續的優化。大部分的編譯器 (我只確定 GCC 和 LLVM) 的 IR 主要都是採 SSA 的形式。

最後，各位要注意 LLVM API 不穩定。看各位是要緊跟 svn 版本進行開發，又或是只跟 release 版本開發。這其中各有利弊，但是別忘了 LLVM 算是很熱心的一個 community， 我想有問題都可以到郵件列表或聊天室詢問。[1][2]

[1] http://lists.cs.uiuc.edu/mailman/listinfo/llvmdev

[2] http://llvm.org/docs/#irc
