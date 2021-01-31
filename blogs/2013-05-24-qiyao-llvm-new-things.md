转换时间： 2021-01-31

# llvm 在尝试的一些新东西
Posted on 2013/05/24 by yao

前几天看到了 llvm 里边的项目都在用buildbot 做 continuous build and test http://lab.llvm.org:8011/ 。continuous build and test 的好处在java 生态系统中已经得到了成功的证明，hudson 或者jenkins 也很成功了。但是continuous build and test 在GNU Toolchain 一直没有得到广泛应用，不知道为什么，没有推广。

今天又看到，llvm的项目开始用 Phabricator 进行review http://llvm.org/docs/Phabricator.html ，感觉在llvm 里边，推广一个新的东西，比较容易，具体原因不知道是什么。 llvm 又比gcc领先了一下。

p.s. llvm 还在用coverage test，也是很先进的测试方式 http://buildd-clang.debian.net/coverage/
