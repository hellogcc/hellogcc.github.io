原始博客链接：

转换时间：
2021-01-31

注意：由于2011年HelloGCC博客遭遇了一次VPS掉电丢失，所有2011年之前（含）的博客都是从 Google Reader 等RSS中恢复的。损失了大部分的内容，非常的遗憾。如果你的电脑中保存过HelloGCC2012年之前的报告，欢迎发送给我们，非常感谢。请转发给 hellogcc.workgroup@gmail.com

# 小例子，函数声明的重要性
Posted on 2011/10/08 by xmj

xmj@hellogcc.org

将函数声明放在头文件里，然后在C文件中包含进来，是一个好的习惯。不然，则可有能导致系统中非常隐蔽的错误。

例子：

```
$ cat prototype.c
int
main (void)
{
 fun (1);
 return 0;
}

void
fun (void)
{
}

$ gcc prototype.c
prototype.c:9: warning: conflicting types for ‘fun’
prototype.c:4: note: previous implicit declaration of ‘fun’ was here
```
这里，gcc只给出了警告信息！然而不匹配的传参操作很可能会造成部分代码（还是数据？）被覆写，程序便会出现非常诡异的现象。


```
$ gcc prototype.c -Werror=implicit-function-declaration
prototype.c: In function ‘main’:
prototype.c:4: error: implicit declaration of function ‘fun’
prototype.c: At top level:
prototype.c:9: warning: conflicting types for ‘fun’
prototype.c:4: note: previous implicit declaration of ‘fun’ was here
```
可以强制将警告信息升级为错误信息来避免这类问题。

后记（来自xunxun）：

可以直接打开-Werror检查所有的warning。

另外，检查语法是否合乎指定的标准(ISO标准)可以用下列开关

检查C99
`-std=c99 -pedantic`
当然可以用
`-pedantic-errors`
不产生警告直接提示错误
