转换时间： 2021-02-01

# 像使用gdb一样，使用perl debugger进行调试
Posted on 2015/12/01 by xmj

调试perl脚本，通过-d选项来启动perl debugger，例如：
```
$ perl -d ./test.pl
```
这里我们先花点时间，把perl的调试环境配置的好使些。

1、增加对Readline的支持

缺省的perl调试器不支持Readline功能，所以，如果你想按up键往前翻历史命令的话，会出现如下乱码：
```
^[[A^[[A^[[A^[[A
```
需要安装模块： Term::ReadKey，Term::ReadLine，Term::ReadLine::Perl（其中后两个我没搞清楚是否重复，反正全装上了）
安装模块的方法，参见 http://www.cpan.org/modules/INSTALL.html

2、使用.perldb文件

这个类似于gdb的.gdbinit，可以用来在调试器启动的时候，做些配置工作，预先执行一些命令。需要注意的是，需要将该文件的权限设定为只有创建者可写，即：
```
-rw-r--r--
```
否则，perl debugger出于安全的考虑，不会去加载这个文件：
```
perldb: Must not source insecure rcfile /home/xmj/.perldb.
        You or the superuser must be the owner, and it must not
        be writable by anyone but its owner.
```
3、打印数组，哈希变量

可以使用Data::Dumper模块，来打印数组，哈希变量的内容，在.perldb中加入如下配置：
```
$DB::alias{'dump'}  = 's/^dump (.*)/p Dumper($1)/';
sub afterinit {
  push @DB::typeahead, "use Data::Dumper qw(Dumper)";
}
```
这样，就可以在perldb中，直接使用dump命令来打印：
```
  DB<10> dump %Options
$VAR1 = 'EnableCheckers';
$VAR2 = [];
$VAR3 = 'DisableCheckers';
$VAR4 = [];
```
4、保存历史命令

在.perldb中加入如下配置：
```
&parse_options("HistFile=$ENV{HOME}/.perldb_hist");
```
下次进入perl debugger的时候，还可以向上翻出早前的历史命令。

———————-
参考链接：
Debugging Perl scripts， http://perlmaven.com/debugging-perl-scripts
Perl Debugger Tutorial: 10 Easy Steps to Debug Perl Program，http://www.thegeekstuff.com/2010/05/perl-debugger/
How can I print the contents of a hash in Perl? http://stackoverflow.com/questions/1162245/how-can-i-print-the-contents-of-a-hash-in-perl
Can the Perl debugger save the ReadLine history to a file? http://stackoverflow.com/questions/6433241/can-the-perl-debugger-save-the-readline-history-to-a-file
How to install CPAN modules，http://www.cpan.org/modules/INSTALL.html
perldebug在线文档：执行命令perldoc perldebug进行查看
