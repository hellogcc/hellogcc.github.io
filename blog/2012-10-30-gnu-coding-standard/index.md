---
title: "GNU coding standard"
date: "2012-10-30"
categories: 
  - "其它技术"
---

刚刚开始GNU 软件开发的人，可能都会被gnu coding standard给搞晕，因为可能自己的编码习惯和这个gnu standard不是很一致，再加上编辑器对tab和space的使用，就使得符合gnu standard变得更加困难。

我个人建议，如果是第一次写gnu的代码，最好去看看gnu standard，不长，

[http://www.gnu.org/prep/standards/standards.html](http://www.gnu.org/prep/standards/standards.html)

还有一个办法，就是写好了代码，用indent去检查一下，

indent -nbad -bap -nbc -bbo -bl -bli2 -bls -ncdb -nce -cp1 -cs -di2 -ndj -nfc1 -nfca -hnl -i2 -ip5 -lp -pcs -psl -nsc -nsob
