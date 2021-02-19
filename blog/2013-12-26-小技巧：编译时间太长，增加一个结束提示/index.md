---
title: "小技巧：编译时间太长，增加一个结束提示"
date: "2013-12-26"
categories: 
  - "其它技术"
---

作者：xmj

由于编译工具一般需要很长的时间，这期间可以做些别的事情，但总是需要不时的检查下是否编译完成。今天总算是找到了我想要的提示功能：

How can I trigger a notification when a job/process ends? http://superuser.com/questions/345447/how-can-i-trigger-a-notification-when-a-job-process-ends

我的做法是，增加一个脚本命令“run”，内容如下：

#! /bin/bash

eval $@
zenity --info --text="Job finished:\\n\\n$\*" --title="Hello"
#notify-send "Job finished:  $\*"

这样，在执行make的时候，运行： $ run make

等执行完，就会自动弹出一个提示窗口了。

另外，我才发现原来ubuntu 12.04 LTS的.bashrc里已经有类似的功能实现了：

\# Add an "alert" alias for long running commands.  Use like so:
#   sleep 10; alert
alias alert='notify-send --urgency=low -i "$(\[ $? = 0 \] && echo terminal || echo error)" "$(history|tail -n1|sed -e '\\''s/^\\s\*\[0-9\]\\+\\s\*//;s/\[;&|\]\\s\*alert$//'\\'')"'
