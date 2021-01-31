转换时间： 2021-01-31

# Clang文档翻译—- Using Clang Tools—-Overview
Posted on 2014/01/10 by shining

原文地址：http://clang.llvm.org/docs/ClangTools.html

译者：shining
简介



Clang工具是为C++开发者所设计的单独的命令行（潜在的图形界面）工具，这些开发者是已经使用Clang并且喜欢使用Clang作为他们的编译器。这些工具提供了面向开发的功能：语法检查，自动格式化，重构等。

只有少数几个最基本和基础的工具被保留在Clang的SVN工程里。剩下的工具都被保存在边上其他的工程里，因为有些用户不想要或者不需要构建他 们。如果你想进入额外的Clang工具代码库，简单的把他们下载到你的Clang的工具目录，然后采用平常所使用的构建过程去和有关联的 LLVM/Clang代码一起构建：
```
    With Subversion:
        cd llvm/tools/clang/tools
        svn cohttp://llvm.org/svn/llvm-project/clang-tools-extra/trunk extra
    Or with Git:
        cd llvm/tools/clang/tools
        git clonehttp://llvm.org/git/clang-tools-extra.gitextra
```
这个文档描述了工程内部的Clang工具的高层次组织的简介，同时介绍了这些工具中更重要的几个。然而，需要注意的是，这个文档只是针对Clang和Clang工具的开发者，不是这些工具的最终用户。
Clang工具的组织

Clang工具是为了让C++开发者可以直接使用的命令行或者图形界面程序。它们最初不是为了给Clang的开发者使用的，尽管希望它们可以对恰巧 工作在Clang的C++开发者有很大帮助，我们积极扩展他们的功能。它们被作为三个部分进行开发：基于Clang而构建的一个单独工具的基础设施；被很 多不同的工具所使用的核心共享逻辑，这些工具是以重构和重写库的形式出现的；工具本身。

Clang工具的基础设施是LibTooling平台。可以从它的文档中看到更多关于它的架构如何工作的细节。重构和重写的工具包风格的库的共同之处也是LibTooling组织的一部分。

几乎没有Clang工具是和核心的Clang库，像基本功能和测试用例那样一起开发。然而，大多数的工具就在边上的代码库（不在项目的代码库）上开 发就是为了证明它们其实就是简单的从核心库中分离。我们没打算支持很多不在项目代码库的公共库，我们想很细心的复审和查找好的库的接口，然后把它们从那些 库里放到核心Clang库系列。

无论Clang的工具的代码放在哪个代码库中，所有的Clang工具的开发过程和实践都是Clang本身的。它们都是Clang项目的一部分，无论使用什么样的版本控制体系。
核心Clang工具

Clang工具的核心系列是在主工程之内的，完成程度已经非常高，并且允许使用和进行Clang的特定功能测试。
clang-check

ClangCheck联合了运行一个Clang工具所需的LibTooling框架和对特定的文件进行快速的语法检查的Clang诊断，使用的是命 令行接口。它也可以通过接受不同的标志来显示不同格式的输出，适合用来驱动IDE或者编辑器。此外，它能在修复模式下直接使用Clang提供的修复提示。 可以从文档How To Setup Clang Tooling For LLVM中去查找如何去建立和使用clang-check的命令。
clang-format

Clang-format既是一个库，也是一个单独的工具，它的目标是根据认证的风格指引去自动格式化C++源码文件。为了达到这个目 标，clang-format使用Clang的Lexer去把一个输入文件转换为一个token流，然后改变这些token周围的的所有空格。 clang-format的这个目标是既要可以作为一个用户工具（理论上拥有强大的IDE集成），同时又要作为其它重构工具的一部分，比如：在重命名的时 候去格式化所有改变的行。
cpp11-migrate

cpp11-migrate 迁移C++的代码去在合适的地方使用C++11的特性。目前它可以：

-    把循环转化为基于范围的for循环；
-    把空指针常量（比如NULL或者0）转换为C++11的nullptr;
-    用auto类型说明符替换变量声明中的类型说明；
-    在适当的成员函数上添加override说明；

额外的Clang工具

不同种类的Clang工具被添加到额外的代码库中，它们将在这里被跟踪记录。这个文档的焦点是对于其他工具开发者来说的工具范围特征，每一个工具都应该提供它自己的聚焦用户的文档。
新工具的思路：

-    C++ cast转换工具。将可以把C风格的casts((type)value) 转换为C++ cast(static_cast,const_cast orreinterpret_cast)。
-    无成员的begin()和end()转换工具。将可以转换foo.begin()为begin(foo)，end()也是同样的，这里的foo是一个标准容器。我们也会探测数组的相似的模式。
-    make_shared /make_unique转换。这个转换器的一部分可以和auto转换器联合起来。它将可以转换：
```
    std::shared_ptr<Foo> sp(new Foo);
    std::unique_ptr<Foo> up(new Foo);

    func(std::shared_ptr<Foo>(new Foo), bar());
```
    变为:
```
    auto sp = std::make_shared<Foo>();
    auto up = std::make_unique<Foo>(); // In C++14 mode.

    // This also affects correctness.  For the cases where bar() throws,
    // make_shared() is safe and the original code may leak.
    func(std::make_shared<Foo>(), bar());
```
-    tr1去除工具。它将使用TR1库的源码迁移到使用C++11的库。例如：
```
    #include <tr1/unordered_map>
    int main()
    {
        std::tr1::unordered_map <int, int> ma;
        std::cout << ma.size () << std::endl;
        return 0;
    }
```
    应该被重写为：
```
    #include <unordered_map>
    int main()
    {
        std::unordered_map <int, int> ma;
        std::cout << ma.size () << std::endl;
        return 0;
    }
```
    一个去除auto工具。将把auto转换为一个明确的类型或者添加推断的类型到注释里。这个工具的动机是因为有开发者不想去使用auto，他们害怕会让他们的代码失控。
    C++14：更少的函数对象冗余操作(N3421)。例如：
```
    sort(v.begin(), v.end(), greater<ValueType>());
```
    应该被重写为：
```
    sort(v.begin(), v.end(), greater<>());
```
