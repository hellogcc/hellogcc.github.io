---
title: "给GDB换一个版本控制工具"
date: "2011-10-18"
categories: 
  - "gdb"
  - "杂谈"
---

最近_Phil Muldoon_在gdb maillist挖了一个大坑 “[GIT and CVS](http://sourceware.org/ml/gdb/2011-10/msg00075.html "GIT and CVS")”，我当时看了以后，觉得这样的话题每过一段时间就会有人提起，每次都因为各种各样的问题，就不了了之了。RedHat他们有自己的一个gdb git branch，叫archer，估计他们每次把git上的patch，commit到cvs上都很郁闷，那个thread里边，Jan （GDB global maintainer）说了，他把git上的12 patch commit到cvs，花了他一个半小时，结果还有两个文件忘记添加了。

我个人觉得git很好，要是能换到git，我也高兴。我现在就之用cvs commit，其他的都不用。谁知道，Eli （GDB global maintainer）说git不好用，而且git不是gnu的项目，她在用bzr，还说“如果我们使用git，这很可能让我gdb里边不活跃了”。后来，楼就歪了，成了比较git和bzr了。里边讨论了很多git bzr很有意思的用法。

git和bzr我都用，感觉bzr就是比git慢一点，其他好像没有什么区别。bzr和launchpad集成很好，不过这些gnu都用不上。

大名鼎鼎的Mark K. (GDB global maintainer)会回复了一次，老外果然说话很直接，也是他的风格 “I am a git hater.”  还列出了他的workflow，他那个workflow是最基本的workflow，就是 update/modify/commit。这样的workflow根本不适合，多个patch的改动，而且在网速慢的地方，就更不合适了。

后来，讨论就没有什么实质进展了，因为没有任何的行动。我觉得这个话题就这样结束了，结果，无意中，在昨天晚上的irc上，看到一些更有意思的讨论。贴在这里：

<tromey\> nobody wants to do the work, just argue about DVCs
<brobecke\> It's OK, though. I think there are disadvantages that are immediately visible as soon as you review a diff, but it's not important enough that I want to pick a fight.
<brobecke\> yeah, me too, sometimes.
<tromey\> ok
<tromey\> it is mostly stop\-energy too
<brobecke\> case in point, the latest discussion about git and bzr...
<tromey\> sometimes I wonder why we put up with it
\* brobecke sighs....
<tromey\> I still haven't read the latest git thread
\* antgreen (~user@c74-230.rim.net) has joined #gdb
<tromey> I didn't want the aggravation
<brobecke\> nothing much there, I wouldn't bother.
<brobecke> there were two threads, really:
<brobecke> (1): what are the problems that need fixing for us to switch to a different VCS
<brobecke> (2): what is the best DVCS?
<brobecke> (1) was a useful reminder, but (2) was a waste of time
<SamB> shouldn't there be a "for us" in #2?
<brobecke\> SamB: Yes, actually there was (a bit)
<SamB\> of course, there's the fact that you are \*already using\* git...
<brobecke> someone even suggested that people who do the most commits should be the ones deciding :-)
<dmalcolm> in case it's useful: http://www.python.org/dev/peps/pep-0374/
<SamB\> brobecke: as someone who has made few or no commits, I am very much in favour of that plan!

这里没有什么好说的，就是他们开始讨论这个话题。第一句是亮点。介绍一下人物吧， brobecke， GDB global maintainer, Release Manager. tromey, GDB global maintainer。好，接着看他么还说什么了。

<tromey\> yeah; but gdb, being a GNU project, has a uniquely bad political atmosphere
<SamB\> indeed
<tromey\> it is something I contemplate quite frequently these days
<tromey\> other fields seem fairer
<SamB\> you guys do \*very\* well, considering
<tromey\> several GNU projects make progress according to the rule of "don't tell RMS"
<tromey\> this works, but really it ought to be beneath us
<brobecke\> If it was just about GDB, I think it would be doable to reach a consensus and just go ahead and do it. But we are intermingled with other projects, and it's costing us big time right now.
<SamB> RMS ought to be saner

不知道这里怎么就开始谴责 GNU和RMS了。

<brobecke\> SamB: the problem is that GDB is part of a larger "project" called src.
<brobecke\> when you checkout gdb, you actually checkout parts of src.
<SamB\> the stupidest name ever
<SamB\> but, yeah, I'm vaguely aware of the CVS repository arrangements
<tromey> binutils guys ought to be on board, since one or two threads ago was on the  binutils list
<tromey> this comes up like every 8 months :)
\* brobecke is setting an alarm, then :)
<tromey> anyway all the commit scripts need to be converted
<tromey> and everything tested
<tromey> and all src communities notified or whatever
<tromey> Joseph posted a bullet list in one of the threads, which was, as usual for him, extremely comprehensive
<tromey> definitive one might say
<brobecke> there are also the "nightly" scripts that create the tarballs, which could use a good rewrite anyway

brobecke解释了为啥把gdb从cvs转换到别的vcs那么困难，说的挺有道理的。

<SamB\> I still don't understand how bzr even qualifies as a GNU project
<tromey> me neither
<SamB> it doesn't seem to have any of the disadvantages usually associated with that status
<brobecke\> why not? (just curious, I never looked at it before)
\* brobecke is reading the PEP document dmalcolm pasted
<SamB\> they'll take my commits without papers, for example
<andre> would a completely separate repo like archer and nightly sync to cvs be an option?
<SamB> It mainly seems to be used to annoy those working on \*actual\* GNU projects by suggesting that they should use bzr
<tromey> I thought bzr required copyright assignment to Canonical
<SamB> for purely political reasons
<SamB> tromey: maybe they do!
<tromey> that for me is a critical flaw
<tromey> I can't imagine what RMS was thinking
<brobecke\> so, to be part of the GNU project, all it takes is RMS accepting it?
<SamB\> anyway, as someone who actually \*likes\* bzr, I'm glad it is not a \*real\* GNU project
<tromey> yes, RMS just has to bless it; but one of the good things about RMS is that he is unusually consistent and principled, so you can be assured it has to be free software at least
<tromey> anyway the bzr decision is one of the things that has really soured me on GNU

这里就有一些有意思的事情了。我以前知道bzr是gnu dVCS，但是不知道参与bzr需要给Canonical 签 copyright assignment。这个是一个很奇怪的事情。community的工作，给一个公司签 assignment，的确很奇怪。

<brobecke\> did he have any alternative, though?
<brobecke\> how does git compare in terms of GNU\-dness, do you think?
<tromey\> this is the thing for me
<tromey\> why does GNU need to bless \*any\* DVC?
<tromey\> choosing bzr does not notably advance the cause of software freedom
<tromey\> there are already many free DVCs
<tromey\> free by every measure that matters to the FSF
<brobecke\> ah, I see.
\* dmalcolm mutters incoherent something about "free-as-in-requiring-copyright-assignment-to-a-for-profit-company"
<tromey\> what also matters to me is (1) the random authoritarianism of RMS \-- it isn't like this was some kind of process like the one Python went to -- and (2) bzr sucks IME; I think GNU should stand for \*both\* software freedom and technical excellence
<tromey> yes, requiring assignment to a company is amazingly bad, especially considering the crap Shuttleworth says about this sort of thing
<tromey> it has been extremely upsetting to me
<tromey> :-(
<brobecke> wow, sorry that it's affected you so much. FWIW, I agree that it should strive for excellence as well.
<SamB\> dmalcolm: hey, it's trivial to fork if at some point they do something evil...
<dmalcolm> one other point about that PEP document: yes, Python does have a "benevolent dictator for life", but the point of the PEP system is to encourage gathering the expert opinion to bear on a subject, so that a decision can be transparently made, and the BDFL is effectively just rubber-stamping it.  It turns the debate from a mailing list thread-of-doom into a deliverable/artefact
<dmalcolm> (sorry to weigh in; waiting on an upgrade here)
<tromey> I would be ok with it if GNU worked this way
<tromey> but RMS is not that open
<tromey> I suppose even if GNU were like this, it would still be dominated by the Eli Zs and Tom Lords of the world and I would still end up looking for something else
<tromey> Apache is perhaps a better model
<tromey> or Fedora or Debian
<pmuldoon> brobecke, did I suggest that? I can't remember what I said ;)
<brobecke\> someone did, not sure who it was.
<pmuldoon\> I think I said my experience was not unique in that the only time I use CVS is when I check\-in
<pmuldoon\> I am sorry about the thread, if it caused problems.  But I feel speaking up is the right thing to do, occasionally, even if it causes headaches ;)
<pmuldoon\> brobecke, and I still think there should some bias to the release maintainer, because that maintainer deals with it
<brobecke\> pmuldoon: thanks! :-). The releases take 2\-3h max twice a year, so the bias should go to heavy contributors.

最后，这样一个讨论也没有什么结论。但是，看上去，换一个vcs，还不是那么简单的事情。我还是不明白，GNU为啥选择bzr，我倒不是说bzr不好，就是觉得那样一中copy right assignment的形式，让别的公司很难接受的。我想这个决定应该RMS做的，不知道他老人家怎么想的。
