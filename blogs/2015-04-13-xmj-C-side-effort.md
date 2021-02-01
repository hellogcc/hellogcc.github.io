转换时间： 2021-02-01

# C语言的副作用
Posted on 2015/04/13 by xmj	

测试例子1：
```
#include

int main(void) {
  int a = 5;
  printf("%d, %d, %d\n", (a++) , (a++) , (a++));
  printf("a = %d\n", a);
  return 0;
}
```
编译执行：
```
$ gcc foo.c
$ ./a.out
7, 6, 5
a = 8
```
测试例子2：
```
#include

int main(void) {
  int a = 5;
  printf("s = %d\n", (a++) + (a++) + (a++));
  printf("a = %d\n", a);
  return 0;
}
```
编译执行：
```
$ gcc-4.7 foo.c
$ ./a.out
s = 15
a = 8

$ gcc-4.8 foo.c
$ ./a.out
s = 18
a = 8```

原因在于C语言标准中关于side effect和sequence point的描述和规范。这里有篇博文，介绍的非常到位（多谢kito-cheng提供）http://blog.tinlans.org/2010/08/06/sequence-point/

C语言标准中的原文如下：
```
The result of the postfix ++ operator is the value of the operand. After the result is
obtained, the value of the operand is incremented. (That is, the value 1 of the appropriate
type is added to it.) See the discussions of additive operators and compound assignment
for information on constraints, types, and conversions and the effects of operations on
pointers. The side effect of updating the stored value of the operand shall occur between
the previous and the next sequence point.

Evaluation of an expression may produce side effects. At
certain specified points in the execution sequence called sequence points, all side effects
of previous evaluations shall be complete and no side effects of subsequent evaluations
shall have taken place.

```
