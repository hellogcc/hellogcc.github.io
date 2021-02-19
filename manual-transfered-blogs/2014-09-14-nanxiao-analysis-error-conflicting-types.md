转换时间： 2021-02-01

# 关于”error conflicting types for function”编译错误的分析
Posted on 2014/09/14 by nanxiao

在使用gcc编译C程序时，有时会碰到“error: conflicting types for ‘function’”的编译错误。从字面意义上理解，是说函数的定义和声明不一致。在这篇文章里，我就对这个错误做个简单的分析（使用的gcc版本是4.9.0）。
（一）首先我们看一个函数的定义和声明不一致的例子：
```
#include <stdio.h>

int func(int a);

int func(void) {
    return 0;
}

int main(void) {

    func();

    return 0;
}
```
编译程序：
```
gcc -g -o a a.c
a.c:5:5: error: conflicting types for ‘func’
 int func(void) {
     ^
a.c:3:5: note: previous declaration of ‘func’ was here
 int func(int a);
```
可以看到由于“func”的声明和定义不一致（一个有参数，一个没有），所以编译时出现了这个错误。

（二）最近我在把一个老程序从Solaris移植到Linux，编译时也出现了这个错误。但是我发现函数在头文件里的声明和函数定义是完全一样的，这就令我很奇怪。查了将近一天时间，最后得到结论是函数参数类型在函数声明后定义了。简化的代码如下：
```
#include <stdio.h>

void func(struct A *A);

struct A {
        int a;
};

void func(struct A *A)
{
        printf("%d", A->a);
}

int main(void) {
        // your code goes here
        struct A a = {4};
        func(&a);
        return 0;
}
```
其中“structure A”的定义放在“func”函数声明之后了，而func函数的参数是“structure A*”类型。编译结果如下：
```
gcc -g -o a a.c
a.c:3:18: warning: ‘struct A’ declared inside parameter list
 void func(struct A *A);
                  ^
a.c:3:18: warning: its scope is only this definition or declaration, which is probably not what you want
a.c:9:6: error: conflicting types for ‘func’
 void func(struct A *A)
      ^
a.c:3:6: note: previous declaration of ‘func’ was here
 void func(struct A *A);
      ^
```
可以看到也输出了“error: conflicting types for ‘func’”的编译错误，也许编译警告可以给一点提示吧。

我检查了一下程序的Makefile，所有编译警告都关掉了，也许是编译警告太多了吧。
