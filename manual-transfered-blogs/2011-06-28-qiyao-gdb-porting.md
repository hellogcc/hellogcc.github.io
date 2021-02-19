原始博客链接：

转换时间：
2021-01-31

注意：由于2011年HelloGCC博客遭遇了一次VPS掉电丢失，所有2011年之前（含）的博客都是从 Google Reader 等RSS中恢复的。损失了大部分的内容，非常的遗憾。如果你的电脑中保存过HelloGCC2012年之前的报告，欢迎发送给我们，非常感谢。请转发给 hellogcc.workgroup@gmail.com

# 探索GDB的移植

Posted on 2011/06/18 by yao

GDB已经被移植到到十几个处理器和平台上，足以证明GDB的结构是便于移植的。本文简单介绍了，作者在移植GDB的过程中，使用和修改的一些GDB hook函数，从而进一步理解这些hook函数的作用。 http://filer.blogbus.com/11149686/resource_11149686_1307586989v.pdf 幻灯片就是上周活动时候用的，加了一点点小的修改。
