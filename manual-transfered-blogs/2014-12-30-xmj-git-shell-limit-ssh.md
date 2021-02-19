转换时间： 2021-02-01

# 使用git-shell来限制用户ssh登陆
Posted on 2014/12/30 by xmj

虽然git是分布式的版本管理系统，但对于团队项目开发，通常还是会在单独的服务器上创建一个 git server。类似于svn，git server 也有好几种配置方式。详情，可以参见git的文档`http://git-scm.com/book/en/v2`。这里，主要是从上述文档中摘出一部分，说明一下git-shell的用处。

假如，大家都使用ssh的方式来访问：
```
$ git clone git@10.3.0.99:project.git
```
这就意味者，访问者具有权限可以ssh登陆到服务器上:
```
$ ssh git@10.3.0.99
```
出于安全的考虑，我们最好限制用户只能进行git push/pull，但无法登陆。这可以使用git-shell来完成。

查看一下git-shell的位置：
```
$ which git-shell
/usr/bin/git-shell
```
将git-shell的路径添加到/etc/shells文件中，然后修改git用户的shell：
```
$ sudo chsh git
```
设置为/usr/bin/git-shell。这样，如果再使用ssh方式登陆，则会报错：
```
$ ssh git@10.3.0.99
git@10.3.0.99's password:
Welcome to Ubuntu 14.04.1 LTS (GNU/Linux 3.13.0-35-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

fatal: Interactive git shell is not enabled.
hint: ~/git-shell-commands should exist and have read and execute access.
Connection to 10.3.0.99 closed.
```
