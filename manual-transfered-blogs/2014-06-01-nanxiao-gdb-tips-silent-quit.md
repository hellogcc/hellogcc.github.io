转换时间： 2021-01-31

# 小技巧：gdb退出时不显示提示信息
Posted on 2014/06/01 by nanxiao

gdb在退出时会提示:
```
        A debugging session is active.

        Inferior 1 [process 19114 ] will be killed.

        Quit anyway? (y or n)
```

如果不想显示这个信息，则可以在gdb中使用`“set confirm off”`命令把提示信息关掉，也可以把这个命令加到`.gdbinit`文件里。
