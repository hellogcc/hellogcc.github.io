---
title: "为td文件生成tags"
date: "2016-05-24"
categories: 
  - "llvm"
---

LLVM的TableGen可以用来为td文件生成tags，方便在vim中查看。该功能的svn信息如下：

\------------------------------------------------------------------------
r177682 | silvas | 2013-03-22 07:40:38 +0800 (Fri, 22 Mar 2013) | 18 lines
Changed paths:
   M /llvm/trunk/utils/TableGen/CMakeLists.txt
   A /llvm/trunk/utils/TableGen/CTagsEmitter.cpp
   M /llvm/trunk/utils/TableGen/TableGen.cpp
   M /llvm/trunk/utils/TableGen/TableGenBackends.h
   A /llvm/trunk/utils/TableGen/tdtags

Add TableGen ctags(1) emitter and helper script.

To use this in conjunction with exuberant ctags to generate a single
combined tags file, run tblgen first and then
  $ ctags --append \[...\]

Since some identifiers have corresponding definitions in C++ code,
it can be useful (if using vim) to also use cscope, and
  :set cscopetagorder=1
so that
  :tag X
will preferentially select the tablegen symbol, while
  :cscope find g X
will always find the C++ symbol.

Patch by Kevin Schoedel!

(a couple small formatting changes courtesy of clang-format)
------------------------------------------------------------------------

使用方法如下：

xmj@xmj-OptiPlex-9020:~/project/llvm-trunk$ bash utils/TableGen/tdtags -q -x all

生成的tags文件：

xmj@xmj-OptiPlex-9020:~/project/llvm-trunk$ find ./ -name tags
./unittests/Option/tags
./lib/IR/tags
./lib/LibDriver/tags
./lib/Target/AVR/tags
./lib/Target/XCore/tags
./lib/Target/WebAssembly/tags
./lib/Target/BPF/tags
./lib/Target/Sparc/tags
./lib/Target/Mips/tags
./lib/Target/NVPTX/tags
./lib/Target/SystemZ/tags
./lib/Target/PowerPC/tags
./lib/Target/Lanai/tags
./lib/Target/MSP430/tags
./lib/Target/X86/tags
./lib/Target/AArch64/tags
./lib/Target/Hexagon/tags
./lib/Target/ARM/tags
./lib/Target/AMDGPU/tags
./test/TableGen/tags
./include/llvm/IR/tags
./include/llvm/Option/tags
./include/llvm/Target/tags
./include/llvm/CodeGen/tags
./tools/clang/tags
./tools/clang/lib/StaticAnalyzer/Checkers/tags
./tools/clang/test/TableGen/tags
./tools/clang/include/clang/AST/tags
./tools/clang/include/clang/Driver/tags
./tools/clang/include/clang/Basic/tags
./tools/clang/include/clang/StaticAnalyzer/Checkers/tags
