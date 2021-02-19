转换时间： 2021-02-01

# 编译gdb注意事项：一定要用一个干净的build文件夹
Posted on 2014/08/03 by nanxiao

上周收到gdb的通知邮件，说gdb 7.8已经release了。作为gdb的忠实用户，自然是体验一下了。我赶紧下了一个版本，然后build了一下，结果在最后link阶段，出了错误：
```
gcc: unrecognized option `-rdynamic'
ld: warning: file /usr/local/lib/libiconv.so: attempted multiple inclusion of file
Undefined                       first referenced
 symbol                             in file
observer_attach_exited              cli-interp.o
observer_notify_signal_exited       infrun.o
observer_attach_sync_execution_done cli-interp.o
observer_attach_command_error       cli-interp.o
observer_notify_signal_received     infrun.o
observer_attach_end_stepping_range  cli-interp.o
observer_attach_no_history          cli-interp.o
ptid_match                          remote.o
observer_notify_end_stepping_range  infrun.o
observer_notify_exited              infrun.o
observer_notify_no_history          infrun.o
observer_attach_signal_exited       cli-interp.o
observer_notify_command_error       event-loop.o
observer_notify_sync_execution_done infrun.o
observer_attach_signal_received     cli-interp.o
ld: fatal: symbol referencing errors. No output written to gdb
collect2: ld returned 1 exit status
Makefile:1334: recipe for target 'gdb' failed
make[2]: *** [gdb] Error 1
make[2]: Leaving directory '/export/home/nan/build_gdb/gdb'
Makefile:8667: recipe for target 'all-gdb' failed
make[1]: *** [all-gdb] Error 2
make[1]: Leaving directory '/export/home/nan/build_gdb'
Makefile:831: recipe for target 'all' failed
make: *** [all] Error 2
```
gdb的编译过程一向是最简单的，link错误我还是第一次碰到。我先是在gdb的IRC里讨论，接着又自己debug，检查代码文件和脚本，结果最后得到的结论是：因为我所用的build目录之前是编译7.7.1版本的，我因为偷懒，没有清空，导致现在编译7.8版本时会有问题，原因应该是link的还是7.7.1的目标文件。

最后得到的经验是：在编译gdb（或是其它软件）时，一定要准备一个干净的build文件夹，否则可能会有你意想不到的问题，像我碰到的在link阶段出问题还好办，如果在执行阶段出了问题，可就更难查了。

注：build文件夹是为了保持源码的干净，通常在源代码文件夹外，新建一个build文件夹，然后所有的configure，make操作等都在这个build文件夹下进行。例如:
```
mkdir build_gdb;
cd build_gdb;
../gdb-7.8/configure
make
make install
```
