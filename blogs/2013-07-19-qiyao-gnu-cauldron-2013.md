转换时间： 2021-01-31

# GNU Tools Cauldron 2013
Posted on 2013/07/19 by yao

https://youtu.be/nSL4jcQCeKg?list=PLsgS8fWwKJZhrjVEN7tsQyj2nLb5z0n70

https://youtu.be/7nfGAbNsEZY?list=PLsgS8fWwKJZhrjVEN7tsQyj2nLb5z0n70


7月12日晚上，注册和欢迎晚宴。在 computer history museum举行。

晚上七点开始。开始以后有各种饮料，大家可以随便聊天，也可以参观这个博物馆。我听说这个博物馆内容挺多的，晚上参观时间有限，而且应该是和之前没有见过的人聊天，而不是去参观。我就提前到了这个博物馆，下午三点多，花了15美元，买票进去参观了，结果它五点就关门，我才看了不到一半。第一个是内容比较多，第二是看英语比较慢。我出来以后，就在附近坐着。
到了6点半以后，参加这次gnu tools cauldron 的人们就陆陆续续来了。很多人都是去年在布拉格见过的，还有一些人是第一次见面。进去注册以后，每个人有一个牌子，上边有名字。特殊的地方有两个，第一，如果你有话题要讲，那么你的牌子上边会有 “Speaker”。第二，需要自己给自己的牌子找一条带子。绿色的带子代表愿意拍照，红色的代表不愿意，还有一个黄色的，代表，照相的时候，需要征求他/她的同意。绝大多数人都是绿色的带子。我看到有一个人挑的红色的。
碰到那个去年在布拉格见过的人，来自intel。随便寒暄了几句，关于reversed debugging，我知道的不多，他也只是用到这个功能，所以就没有说太多。 我的牌子上有 “Speaker”，还挺好。很多人看到这个，都会问你，要讲什么。碰到了一个日本老先生，很友好，问了问我的话题。他来这里，就是想看看如何用gcc plugin 做一些编码规范的检查。他对中国了解很多，让我惊讶，他知道西安，武汉。要结束的时候，碰到了Tom Tromey，GDB global maintainer，之前从来没有见过。他和我老板是老熟人了，寒暄了几句，我也没有说话。晚上就回去了。

7月13日，gnu cauldron 就开始了。早上的一个小插曲是，班车司机把我们放到了错误的google building。开会在google 的一个 building，距离酒店可能有4公里的样子。班车司机把我们放到一个building 门口就走了，我们发现位置不对。还好 google compus 里边很多 google bikes，我们30来人就骑车到了正确的building。

第一次进入google building，茶水间不错，会议室也很好，不过也没有想象的那么好。有几个冰柜，里边有饮料和冰激凌等等。我们用了其中两个会议室。很快第一个session 就开始了。

第一个session 是 GNU Toolchain ecosystem on AIX。是 David Edelsohn 讲的。主要介绍了AIX 的一些特殊之处和Toolchain做的一些改变。里边关于 shared archive 的问题我之前还碰到过。 AIX 是 SysV R3，而Linux 是 SysV R4。所以在共享库的实现有些不一样。也不是很清楚，他介绍这些的目的是什么，就是科普的介绍一下吧。 David Edelsohn 是 GCC steering committee的，属于资格很老的人啦。
第二session就是我的， Port GDB to a new architecture processor: TI C6x。这个话题和我在去年hellogcc上讲的类似，有一些调整。比如把一些基本知识删除掉了，因为来到cauldron 的应该都是很了解这些基本知识的。虽然英语不好吧，但是也不怎么紧张。被一个老头问了几个问题，后来才知道是 Michael Eager，他维护的 DWARF。讲的怎么样，我不好说，想知道的自己看视频吧。

:)下来的就是 teawater 的 kgtp了。我刚讲完，比较累，没有仔细听。想知道的自己看视频吧。

上午的所有就结束了，进入了午饭时间。午饭很简单，就是一些三明治和沙拉，没有在布拉格吃的好。

中午吃饭的时候，和几个cisco 的人做在了一起。我突然想起来，在我刚刚去codesourcery的时候，我就听说 Jim Wilson 离开了 codesourcery，去了cisco。我就提了一句，结果他还来了，一个很腼腆的老先生， 也是 steering committee的。很牛的人。
下午的第一个session 是 Tool for Automatic Compiler Tuning。是俄罗斯一个研究院做的，是三星赞助的，target 是arm。他们的思路比较有意思，用 Genetic Algorithm， 排列所有的gcc 的options，然后跑一些benchmark，看看那些option 对性能有贡献，有些没有贡献。有些结论很有意思，大多数opton 对程序的性能是没有影响的，只有 15 – 30% 的 option 是有影响的。
第二个session 是GNU C Library。session 实在太让我失望了，唯一的收获就是看到了google 会议室里边，如何进行视频会议。有一个 glibc 的重要maintainer 没有办法参加会议，所以他们采用视频会议的方式。其实就是google+上的视频功能。有一个网页，是大家可以共同编辑的白板。其实就是一个很自由的讨论。讨论的一个问题，就把我吓倒了，尽然是关于怎么用bugzilla 来控制 glibc 的release。release 之前看看有没有p1的bug，我的天，glibc 都20多年了，都没有这样的控制管理吗？就是觉得他们讨论的问题，都很无聊。
第三个session是 Accelerator。其实就是有三组人想给gcc里边就加入一些 为了并行的语言扩展。redhat 是做openmp，最近的gcc patches 里边看到了一些patch。intel 他们也有自己的interface，针对异构多处理器的。我们mentorgraphics，想做一些 openacc 的东西。讨论主要就是看看，这个三组effort 能不能share 一些 infrastructure in gcc。
第四个session 是 autofdo，一个华人做的。我们之前在gcc patches 里边，也看到过他的patch。思路很直接，通过profile 的结果来指导优化。只能在x86上使用。听上去，实现的非常好。
第五个session 是 GCC-plugin for light-weight bounds checking。一个大学做的，纯粹的research 项目。效果听着不错，但是不知道他在gcc里边做了什么。印度口音有些重，我没怎么听懂。
晚上Mark Mitchell 在 San Jose 请我们吃饭，一个法国餐厅，因为法国国庆节，还有special menu，不过没有觉得有多好吃。没有参加cauldron 的晚餐。有些小小的遗憾。

7月14号。Steering Committee / Release Managers。就是让四个steering committee members 坐在前边，接受大家的问题。有人问，是否考虑加入一些 新鲜血液，回答是yes。但是至于怎么加入，没有讨论，和没有说一样。还有人提到了，copyright assignment 的流程怎么那么慢，他们也不知道。还有人建议，可以在fsf foundation 里边，为gcc 专门再建立一个 foundation，我觉得好不靠谱呀。
Integrate Data Race Detection into GDB。 这个session 是一个intel 的 德国工程师Markus做的。大概框架就是，icc 可以在编译源代码的时候，做instrument，然后用自己的lib 在运行时检测data race。他们的工作就是当race 出现的时候，在race 发生的地方，设置断点，然后当race 发生的时候，程序自动停止下来，可以用gdb 检查一下程序的状态。我之前做过一些race decection  的工作，所以和聊了很多。很不错的工作，不过绝大部分都没有 upstreams。
ARM/AArch64 BoF。这个session的第一个收获就是第一次听一个外国人念 AArch64，A-Arch-64。对整个arm 的工作没有什么了解，但是对arm shanghai team 的工作还是有些了解的，发现这个session 里边也提到了很多他们的工作，比如ivopt，newlib-nano, conditional comparison等等，我为他们感到骄傲， Good job。
下午还有一个session是 DejaGNU 和 testing。 Rob Savoye 讲的。之前吃早饭的时候，听过他说话，听着就不一般。打扮就更不一般了，牛仔的打扮。听他的session 的时候，就知道，他是main maintainer of dejagnu and gnash。晚上把他的名字在google 里边搜索了一下，我就傻了，好牛的人。他是cygnus 的第一批员工。剩下的就看wikipedia 和 他的website吧。开会的结论就是需要为gdb 对 dejagnu 做一些修改，提出了很多wild 的 想法，至于能改成什么样子，就不知道了。 对了，Rob 现在被 Linaro hire了，做一些arm performance tuning 的工作。

最后就到了wrap up了，和往常一样，讨论下次在哪里开，提议欧洲，我举手赞成，后来又有人提议亚洲，我举双手赞成。然后就是和各个认识的人，握手告别。
