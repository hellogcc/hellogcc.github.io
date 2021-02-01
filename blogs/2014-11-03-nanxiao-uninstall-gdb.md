转换时间： 2021-02-01

# 如何卸载gdb？
Posted on 2014/11/03 by nanxiao

前几天安装最新的 `gdb` 过程中出了点问题，想卸载一下。结果执行“`make uninstall`”命令后，出现下面的结果：
```
bash-3.2# make uninstall
the uninstall target is not supported in this tree
```
看起来，gdb不支持直接用“`make uninstall`”命令卸载，那么如何卸载它呢？我在`gdb mailing list`里问了一下，收到了 Doug Evans 的回信：
```
Yikes.

A clumsy workaround is to cd into each subdir in the build tree and do
make uninstall there.
```
只能是进入每个子目录，分别执行“`make uninstall`”命令了。

后来，他又在一封单独的信中提到，他已经提交了bug，看看会不会有人做一下改进了。
