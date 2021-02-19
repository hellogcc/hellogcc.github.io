---
title: "小例子，函数声明的重要性"
date: "2011-10-08"
categories: 
  - "其它技术"
---

xmj@hellogcc.org

将函数声明放在头文件里，然后在C文件中包含进来，是一个好的习惯。不然，则可有能导致系统中非常隐蔽的错误。

例子：

<table><tbody><tr><td><pre>1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
</pre></td><td><pre style="font-family:monospace">$ cat prototype.<span style="color:#202020">c</span>
<span style="color:#993333">int</span>
main <span style="color:#009900">(</span><span style="color:#993333">void</span><span style="color:#009900">)</span>
<span style="color:#009900">{</span>
 fun <span style="color:#009900">(</span><span style="color:#0000dd">1</span><span style="color:#009900">)</span><span style="color:#339933">;</span>
 <span style="color:#b1b100">return</span> <span style="color:#0000dd">0</span><span style="color:#339933">;</span>
<span style="color:#009900">}</span>
&nbsp;
<span style="color:#993333">void</span>
fun <span style="color:#009900">(</span><span style="color:#993333">void</span><span style="color:#009900">)</span>
<span style="color:#009900">{</span>
<span style="color:#009900">}</span>
&nbsp;
$ gcc prototype.<span style="color:#202020">c</span>
prototype.<span style="color:#202020">c</span><span style="color:#339933">:</span><span style="color:#0000dd">9</span><span style="color:#339933">:</span> warning<span style="color:#339933">:</span> conflicting types <span style="color:#b1b100">for</span> ‘fun’
prototype.<span style="color:#202020">c</span><span style="color:#339933">:</span><span style="color:#0000dd">4</span><span style="color:#339933">:</span> note<span style="color:#339933">:</span> previous implicit declaration of ‘fun’ was here</pre></td></tr></tbody></table>

这里，gcc只给出了警告信息！然而不匹配的传参操作很可能会造成部分代码（还是数据？）被覆写，程序便会出现非常诡异的现象。

<table><tbody><tr><td><pre>1
2
3
4
5
6
</pre></td><td><pre style="font-family:monospace">$ gcc prototype.<span style="color:#202020">c</span> <span style="color:#339933">-</span>Werror<span style="color:#339933">=</span>implicit<span style="color:#339933">-</span>function<span style="color:#339933">-</span>declaration
prototype.<span style="color:#202020">c</span><span style="color:#339933">:</span> In <span style="color:#000000;font-weight:bold">function</span> ‘main’<span style="color:#339933">:</span>
prototype.<span style="color:#202020">c</span><span style="color:#339933">:</span><span style="color:#0000dd">4</span><span style="color:#339933">:</span> error<span style="color:#339933">:</span> implicit declaration of <span style="color:#000000;font-weight:bold">function</span> ‘fun’
prototype.<span style="color:#202020">c</span><span style="color:#339933">:</span> At top level<span style="color:#339933">:</span>
prototype.<span style="color:#202020">c</span><span style="color:#339933">:</span><span style="color:#0000dd">9</span><span style="color:#339933">:</span> warning<span style="color:#339933">:</span> conflicting types <span style="color:#b1b100">for</span> ‘fun’
prototype.<span style="color:#202020">c</span><span style="color:#339933">:</span><span style="color:#0000dd">4</span><span style="color:#339933">:</span> note<span style="color:#339933">:</span> previous implicit declaration of ‘fun’ was here</pre></td></tr></tbody></table>

可以强制将警告信息升级为错误信息来避免这类问题。

后记（来自xunxun）：

可以直接打开-Werror检查所有的warning。

另外，检查语法是否合乎指定的标准(ISO标准)可以用下列开关

检查C99  
\-std=c99 -pedantic  
当然可以用  
\-pedantic-errors  
不产生警告直接提示错误
