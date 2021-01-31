转换时间： 2021-01-31

# Side effect analysis in LLVM
Posted on 2014/05/30 by xmj

简单介绍下LLVM中的副作用分析。关于LLVM的别名分析基础设施，可以参考：http://llvm.org/docs/AliasAnalysis.html。

何为副作用分析，简单来讲，就是分析执行一条语句（包括函数调用）是否会对内存对象有读写操作（Mod/Ref）。书上（包括龙书等）讲的大多都是fortran时代的算法，而对于c语言这样的，由于引入了指针，使得情况变得复杂了。所以，后来的论文大多集中在别名分析或指向分析领域。副作用分析可以看作是别名分析的客户程序，指针分析搞定，副作用分析就迎刃而解。

LLVM中，在2.6以及之前，是有Andersen别名分析的实现的，后来因为无人维护，有许多问题就不再支持。在单独的一个项目poolalloc中，有对DSA（data structure analysis）的实现，以及基于DSA的Steensgaard别名分析的实现，这部分正是Chris Lattner的博士论文所讲的内容。然而真正属于LLVM内核代码中的，主要也就是basicaa和globalsmodref-aa，这些都是比较简单但有效的实现。gcc也是如此，产品级的编译器，会综合考虑精度和开销的问题，并对健壮性有更高的要求。通常，我们说副作用分析是以别名分析为基础的，但是globalsmodref-aa恰恰反过来，它是先分析globals的mod/ref信息，然后推导别名信息。

LLVM中，对于普通的语句（非函数调用），其副作用可以直接通过别名分析的结果推导出来，例如：
```
AliasAnalysis::ModRefResult
AliasAnalysis::getModRefInfo(const LoadInst *L, const Location &Loc) {
// Be conservative in the face of volatile/atomic.
if (!L->isUnordered())
return ModRef;

// If the load address doesn't alias the given address, it doesn't read
// or write the specified memory.
if (!alias(getLocation(L), Loc))
return NoModRef;

// Otherwise, a load just reads.
return Ref;
}
```
对于函数调用的情况，需要额外处理。所以对于一个新的继承自AliasAnalysis类的别名分析实现，通常会将其作为接口单独实现。

LLVM中全局变量，函数都是指针类型的，其指向对应的内存分配。对于c语言“a = b + c；”，我们可能会说这条语句引用了b和c，修改了a。但，其翻译到LLVM中间代码的时候，已经是多条语句，包含了load，add，store。所以，对于LLVM的中间语言，add是不涉及到副作用的，只有load和store这样的对内存有操作的，才具有副作用，正如代码所示，

```
ModRefResult getModRefInfo(const Instruction *I,
const Location &Loc) {
switch (I->getOpcode()) {
case Instruction::VAArg: return getModRefInfo((const VAArgInst*)I, Loc);
case Instruction::Load: return getModRefInfo((const LoadInst*)I, Loc);
case Instruction::Store: return getModRefInfo((const StoreInst*)I, Loc);
case Instruction::Fence: return getModRefInfo((const FenceInst*)I, Loc);
case Instruction::AtomicCmpXchg:
return getModRefInfo((const AtomicCmpXchgInst*)I, Loc);
case Instruction::AtomicRMW:
return getModRefInfo((const AtomicRMWInst*)I, Loc);
case Instruction::Call: return getModRefInfo((const CallInst*)I, Loc);
case Instruction::Invoke: return getModRefInfo((const InvokeInst*)I,Loc);
default: return NoModRef;
}
}
```
副作用分析的客户程序又是哪些呢？这包括：`-adce`（Aggressive Dead Code Elimination），`-licm`（Loop Invariant Code Motion），`-gvn`（global value numbering）等。看起来，如果副作用分析做好的话，还是会有性能提升的。

副作用分析的效果又如何呢？实际情况中，往往会有函数调用了外部函数（比如c库），为了正确性，必须保守的认为其对所有的内存对象都有副作用。这就导致整个调用链上的函数都将保守处理，以至于分析精度严重受损，效果也会大打折扣。所以LTO很关键，但是实际情况中，对于c库之类的，我们还是用的二进制包，LTO也无能为力。
