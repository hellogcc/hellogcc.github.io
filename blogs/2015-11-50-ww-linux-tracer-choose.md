转换时间： 2021-02-01

# Linux下那么多Tracer如何选择
Posted on 2015/11/30 by ww

如果你想要在linux下调调kernel, 抓抓程序的性能, 那么首先想到的可能是 OProfile 和 Linux Perf. 但是显然, 开源有一个非常显著地你无法回避的特点, 就是你会有太多的选择: perf, oprofile, systemtap, dtrace4linux, lttng, kgtp, ktap, sysdig, ftrace, eBPF. 是不是已经眼花了? 那么你不能错过这篇文章:

http://www.brendangregg.com/blog/2015-07-08/choosing-a-linux-tracer.html

介绍了目前常用的各类工具:
```
    ftrace
    perf_events
    eBPF
    SystemTap
    LTTng
    ktap
    dtrace4linux
    OL DTrace
    sysdig
```
作者非常细心的列出了大量的工具原理及使用教程, 保证会花掉你大把的晚上才能看完.

其实我日常使用的只有 perf, 只能算是 `‘most people’`
