原始博客链接：

转换时间：
2021-01-31

注意：由于2011年HelloGCC博客遭遇了一次VPS掉电丢失，所有2011年之前（含）的博客都是从 Google Reader 等RSS中恢复的。损失了大部分的内容，非常的遗憾。如果你的电脑中保存过HelloGCC2012年之前的报告，欢迎发送给我们，非常感谢。请转发给 hellogcc.workgroup@gmail.com

# 查看struct gcc_target的定义
Posted on 2011/06/22 by xmj

xmj@hellogcc 现在结构体struct gcc_target的定义，是通过一系列的宏来实现了，相关文件包括：target.h，target.def，target-hooks-def.h。这些宏，包括 #define DEFHOOKPOD(NAME, DOC, TYPE, INIT) TYPE NAME; #define DEFHOOK(NAME, DOC, TYPE, PARAMS, INIT) TYPE (* NAME) PARAMS; #define HOOK_VECTOR_1(NAME, FRAGMENT) HOOKSTRUCT(FRAGMENT) #define HOOK_VECTOR(INIT_NAME, SNAME) HOOK_VECTOR_1 (INIT_NAME, struct SNAME {) #define HOOK_VECTOR_END(DECL_NAME) HOOK_VECTOR_1(,} DECL_NAME ; ) 。。。 如果，要实现，增加一个target hook，就需要在target.def文件里增加相应的条目。为了能够查看实际的结构体定义，可以在trunk/gcc目录下，运行命令 $ gcc -E target.h -I ../../build-trunk/gcc/ -o ~/temp/target.i 查看~/temp/target.i文件，可以看到struct gcc_target的定义： struct gcc_target { struct […]
