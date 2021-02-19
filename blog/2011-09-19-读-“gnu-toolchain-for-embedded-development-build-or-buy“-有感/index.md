---
title: "读 “GNU Toolchain for Embedded Development: Build or Buy?“ 有感"
date: "2011-09-19"
categories: 
  - "杂谈"
---

这个题目是在有点老套了，让我想起了初中时代，写作文的题目。今天无意中看到了这样一个white paper，Mark Mitchell写的。看了一遍，还是有些思维的跳跃，跟大家分享一下。原文 “[GNU Toolchain : Build or Buy?](http://www.mentor.com/embedded-software/resources/overview/gnu-toolchain-for-embedded-development-build-or-buy-5ad09fc9-82d2-49e8-88cc-15ad16ea5012)” 需要注册下载。

我在这里也不评论到底是应该build还是应该buy，这个结论也不是我这个文章的重点。我就是想看看从这篇文章里边，我们能知道一些GNU toolchain开发的状况。

这篇文章14页，不长。跳过前边，直接从第四页开始，介绍了Building The Toolchain，这个是一个复杂的过程。自己build 过 toolchain的人都应该有这样的感觉，

需要了解每个toolchain component的版本。有些版本的某个component可能有bug，或者某两个component在某个两个版本上，不能一起使用，等等。这些问题需要在一开始就知道。

应用哪些patch？toolchain component可能有问题，所以在某个版本后，才会有相应的patch fix，那么那些patch应该用，用了以后会不会带来新的问题，等等，这些问题也要考虑。

configure的选项。虽然几乎所有的gnu toolchain component都是遵循 configure make make install的过程，但是每个都有十分复杂的configure 选项。

上边这些无非是告诉我们，build toolchain就是一个十分复杂的事情。但是我们可以看出，这些东西从侧面告诉我们，这些就是一个依靠GNU toolchain生存的公司或者部门的必须知道或者精通的知识和技能。之前和朋友聊过，说，我们什么时候也能开一个toolchain 的公司，我们当时开玩笑说，等有了global maintainer再说吧。其实上边这些就是开toolchain公司的一些必要条件。

文章接下来讲了toolchain validation。里边很多都是介绍GNU toolchain怎么测试的，我这里就跳过了。有意思的部分是“分析和修正test failure”。对于toolchain这样的严格系统软件，很多时候，是一种 “bug-fix-driven development”，每一个release至少要保证没有严重的错误，然后再保证一些性能的提高或者新的特性。所以，在toolchain这个行当里边，“修bug”比别的行当重要的多。在我之前的工作中，修bug好像是一个很没有技术含量的事情，可能因为那些bug也没有什么技术含量。GNU toolchain的工作，很大部分是在修bug，而且我现在认为，修bug是进步最快的一种方式。

好，回到toolchain validation，在测试中发现了fail，就需要fix。文章中给出了一个最“乐观”的方式，定位出问题的component，理解代码，发问题到list，自己修正，把patch发送到list。但是，实际上，这样的方式对大多数人，行不通的。就“理解代码”这样个事情，就很复杂，toolchain的代码量那么大，而且之间还有一些交互，理解代码是很难的事情。 总之，能够自如的使用那个最“乐观”的方式，是自如的加入GNU toolchain开发的一个必要条件吧。

希望这样一个文章不会让我们这样在toolchain行当里边挣扎的人感到绝望，相反，希望我们能看到一些曙光，希望和方向。我看到了，你呢？ :)
