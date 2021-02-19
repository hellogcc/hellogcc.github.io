---
title: "HelloGCC 2011 Workshop 见闻"
date: "2011-09-30"
categories: 
  - "社区活动"
---

HelloGCC是下午1点30开始，arm的四位同事是从上海飞过来的，十点半到的北京，我们一起在物科宾馆吃午饭，这个宾馆的餐厅很不错，午饭非常愉快。

他们team来了四个。其中有两个人都是之前在maillist见的，今天终于见到真人了。不过，他们见到了teawater真人的时候，还是比较激动的，都用了一句 “久闻大名”，见到我的时候，都来了一句“哦，你叫 qi yao，不叫 yao qi啊”，好像很多人第一次见我，都会有这样一句。后来吃饭lingkun和liujia也来了，linkgun帮我们贴了很多引导箭头，实在也辛苦了。

话题范围也比较广，从京沪高铁到arm maintainer；从linaro到auto vecterization和八卦出来的那个以色列美女； 吃饭吃到一半，teawater和ericfisher去会场准备去了，我们剩下的接着吃饭，这里感谢一下他俩。这里还有负责会场的ChinaUnix的老周，十分敬业。

[![](images/204451t-1167338376_d.jpg "教室门口签到")](http://www.hellogcc.org/wp-content/uploads/2011/09/204451t-1167338376_d.jpg)  
吃完饭，我们一起到了会场，也就基本没有什么事情了。ericfisher和pzhao在准备开场介绍，方方面面都要考虑到，不容易的。

一点半，活动开始，我们的主持人pzhao上去介绍了一下我们的活动，一开始有点紧张，后来就好了。接下来，是中科院开源协会的讲话，一会就讲完了。

[![](images/204451t921118917_d.jpg "主持人开场介绍")](http://www.hellogcc.org/wp-content/uploads/2011/09/204451t921118917_d.jpg)

我们的话题就正式开始了，

第一个话题 介绍gcc后端的工作流程。没有废话，直接进入主题。感觉缺少一点outline。可能听众都不太清楚这个话题的提纲是什么。第一页，非常有杀伤力，这样讲gcc的，绝对体现大牛实力。用了一个很好的图来介绍gcc内部的流程和结构。如果没有gcc后端的经验，可能不好理解。这个slide很赞的地方就是用一个简单的例子来介绍gcc各个阶段的不同表示。rtl pattern太小了，我在第一排，我看不清。  
生成的rtl依然看不清。中间有一页是关于move合法性，我个人对这个关于move的内容很有兴趣，不过有点遗憾，没有在这个上边讲太多内容。 后来，那个字典比喻很好。关于move，也许这里还有更多的东西，我们可以深入研究一下。

 [![](images/204451t1084451443_d1.jpg "第一个话题")](http://www.hellogcc.org/wp-content/uploads/2011/09/204451t1084451443_d1.jpg)

顺便说一句，中间有一个女孩特意跑来让我们提示演讲者，不要离话筒太近，不然音响效果不好，都听到喘气声了。这里感谢一下那位女孩。

slides突然跳到了llvm感觉很突兀。后边的很精彩。总体来说，技术内容十分充足，但是表达方式需要改进。slides被有些生硬的划分成了若干部分，除了我们几个经常一起交流的，可能其他人很难把这些部分，联系在一起吧。

第二个话题是 arm 上海的Joey Ye讲的。 一个很不错的开场”good news vs bad news”。对他们每个team成员介绍。

[![](images/204451t-80873275_d.jpg "第二个话题")](http://www.hellogcc.org/wp-content/uploads/2011/09/204451t-80873275_d.jpg)  
他对arm toolchain 进行了介绍，也许是我的背景比较了解吧，所以感觉一切都很自然，很容易理解。然后他介绍了，他们那个gcc branch和gcc mainline以及gcc 4.6 branch的关系，是一个很好的例子。国内，以前也没有人说过如果自己做一个branch，patch应该是怎样merge的。这个我也觉得很熟悉。我在这里简单说一点吧，关于自己的branch。gcc的开发模式，是这样的，比如gcc 4.6 branch还在，同时gcc mainline （4.7）也在开发，但是mainline还没有release过，所以，一般没有人在mainline build 然后release给自己的客户。好的方式是在gcc 4.6 branch的基础上，创建自己的branch。这里，自己的branch的原则是，尽可能少的携带自己local的patch，除非自己的branch十分需要，但是upstreams （gcc 4.6 branch 或者 gcc trunk 4.7）不要这个patch的情况。所以，一旦在自己的branch 有了patch，首先想到的是，contribute到mainline，如果不适合mainline，那么就要试图contribute到gcc 4.6。这样，gcc 4.6 branch接受了这个patch后，就可把这个patch backport到自己的branch。基本上，所有的branch都是 这样run的。

中间讲到了一个如何从头开始build armtoolchain的slide很有意思。应该是在什么地方得到他们的build文档和脚本，然后可以从一个新的ubuntu的环境中，按照文档中描述，build出一个arm cross toolchain的binary。这样很好，正如“Build or Buy？”讲到的，build自己的toolchain是十分复杂的事情，能有这样的文档和脚本，的确很好。

第三个话题是关于displaced stepping的。可能大家对这个领域不是很熟悉，所以没有什么太多的问题。对过程中，那个板凳的比喻印象深刻。

 [![](images/204451t-1694539531_d.jpg "第三个话题")](http://www.hellogcc.org/wp-content/uploads/2011/09/204451t-1694539531_d.jpg)

第四个话题是关于llvm的。讲得很好，很清楚，一看就是做学问的。对自己的工作和别人的工作都讲述的十分客观。犹豫我对背景没有什么了解，问了一些很基本的问题。里边吸引我的地方就是可以为自己的arch在llvm中添加pass，来达到自己的目的。我觉得这个很好。本来想问他们那些工作是否曾经尝试提交到upstream，后来记了。我觉得是很好的工作，如果愿意贡献出来，也是我们国人的骄傲。

[![](images/204451t2028238485_d.jpg "第四个话题")](http://www.hellogcc.org/wp-content/uploads/2011/09/204451t2028238485_d.jpg)

在benchmark的图表上，要是能再细致一点，把那个总计时间，分开，这样比较起来更有指导意义一些。

第五个话题是关于 gcc plugin。  
对插件的历史和之前的一些争论做了很好的总结和评价。然后就在介绍他自己做的那个gcc 插件，台下掌声不断。的确是我们的榜样。其中一个人的问题很好，关于vcg是否能用dot生成图像，现在vcg还没有。不过我个人觉得dot是一个很简单的语言，而且它的layout做的很好，不过作者的担心是dot只能编译成为pdf或者jpeg才能看，没有直接浏览的办法。

[![](images/204451t-948431724_d.jpg "第五个话题")](http://www.hellogcc.org/wp-content/uploads/2011/09/204451t-948431724_d.jpg)

最后的问题，现在很多插件还都是处在一个玩具的状态，好像没有一个成熟的插件。的确现在是这个，我觉得，gcc的版本变化造成了plugin的接口变化，这样的一个趋势导致没有人敢做插件，因为接口总在变。

个人看法：gcc community还在犹豫，就是又想打造很好的生态系统，让别人来参与，但是又担心，别人用了gcc的功能去做自己的商业工具。我个人一直不同意插件需要gpl。之前gnu的软件使用gpl，能够成功，是因为软件及其复杂，比如gcc或者gdb，即使别人能有代码，依然有很多别的技术门槛使得只有source cdoe是无法工作的。就像”Build or Buy?”那个文章里边讲到的。但是gcc plugin都不会很复杂，对于一个不是很复杂的软件，如果公开了源代码，就没有任何的商业价值了。不过，GCC作为GNU和GPL的标杆性项目，是一个很重要的阵地，所有人都比较慎重，保守一点可以理解。

最后到了抽奖，两个pad的大奖。我还真想要个pad，但是抽奖又没有我。不过还是恭喜那两位幸运听众。我们的主持人pzhao还是大度的，自己被抽到了，但是还是把机会让给了别人，值得表扬。 hellogcc 2011会场部分也就这样结束了。有点小小如释重负的感觉。

[![](images/204451t-1762102677_d.jpg "书籍抽奖")](http://www.hellogcc.org/wp-content/uploads/2011/09/204451t-1762102677_d.jpg)

人都散去了以后，我们合影留念，我们私下聊天，交流一些做GNU toolchain的心得，很不错。我和arm的Terry Guo 聊了很多。然后我们建议大家一起留影纪念。

[![](images/204451t-1550277399_d.jpg "HelloGCC 2011合影")](http://www.hellogcc.org/wp-content/uploads/2011/09/204451t-1550277399_d.jpg)

第一排从左到右，依次是 chengbin，ericfisher，yao，teawater，pzhao，老周（ChinaUnix)。第二排从左到右依次是lingkun，wuwei，liujiangning，Joey Ye, 不认识，xuguowei，不认识，Jia，Terry Guo。

然后我们就一起去吃晚饭。workshop已经圆满结束了，我们好像更有心思聊天了。当然，只要teawater在，我们都总有话说。我们聊了聊某位gdb maintainer总在用一种没有任何建设性意见的方式去review别人的patch，这样的确很不友好。大家还对明年的活动提出了各种各样的想法，很有意思。最后，吃饭完毕，都到了晚上九点了，一天都过去了。HelloGCC workshop 2011就这样结束了。
