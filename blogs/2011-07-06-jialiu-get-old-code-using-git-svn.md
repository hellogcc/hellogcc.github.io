原始博客链接：

转换时间：
2021-01-31

注意：由于2011年HelloGCC博客遭遇了一次VPS掉电丢失，所有2011年之前（含）的博客都是从 Google Reader 等RSS中恢复的。损失了大部分的内容，非常的遗憾。如果你的电脑中保存过HelloGCC2012年之前的报告，欢迎发送给我们，非常感谢。请转发给 hellogcc.workgroup@gmail.com

# 使用git-svn获取历史悠久的代码
Posted on 2011/07/06 by admin

Jia.Liu@hellogcc

这年头你不git好像都不好意思跟人打招呼。

gcc是个svn项目，但是有git-svn可以用。

```
git svn clone svn://gcc.gnu.org/svn/gcc/trunk gcc
```

就可以clone出来了，以后你本地就是个git项目了。

但是，这样clone会非常非常慢，因为git需要获取版本信息，现在gcc这么多年了，版本信息？我估计你clone一星期能clone完就不错。

所以，我们可以不要那些很久之前的版本信息，先运行

```
svn info svn://gcc.gnu.org/svn/gcc/trunk
```

查看当前的版本号是多少。

```
Path: trunk
URL: svn://gcc.gnu.org/svn/gcc/trunk
Repository Root: svn://gcc.gnu.org/svn/gcc
Repository UUID: 138bc75d-0d04-0410-961f-82ee72b054a4
Revision: 175617
Node Kind: directory
Last Changed Author: gccadmin
Last Changed Rev: 175616
Last Changed Date: 2011-06-29 08:18:52 +0800 (Wed, 29 Jun 2011)
```

比如这个是175616，那么我们就可以运行
```
git svn clone -r175615:HEAD svn://gcc.gnu.org/svn/gcc/trunk gcc
```

来clone了。

！！！注意！！！
你指定的版本一定要小于175616，等于都不行！所以我用了-r175615。

该方法不适合考古人士。
