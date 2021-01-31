原始博客链接：

转换时间：
2021-01-31

注意：由于2011年HelloGCC博客遭遇了一次VPS掉电丢失，所有2011年之前（含）的博客都是从 Google Reader 等RSS中恢复的。损失了大部分的内容，非常的遗憾。如果你的电脑中保存过HelloGCC2012年之前的报告，欢迎发送给我们，非常感谢。请转发给 hellogcc.workgroup@gmail.com

# IRC聊天随笔
Posted on 2011/11/25 by xmj

1.查看eh_frame的内容。

$ readelf -wF foo

2.gdb里查看段信息。
```
(gdb) info files
```
3.记录应用程序执行的PC值
```
qemu-i386 -d in_asm foo
```
