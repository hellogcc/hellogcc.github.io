---
title: "回路搜索算法"
date: "2016-05-04"
categories: 
  - "gcc"
  - "llvm"
---

Gcc和LLVM中的软流水优化，都是基于Swing Modulo Scheduling算法（翻译成摆动模调度？）实现的，参见\[1\]。其中，需要在依赖图（dependency graph）上搜索所有的简单回路（elementary circuits）。

LLVM中的软流水代码里，提到了“Identify all the elementary circuits in the dependence graph using Johnson's circuit algorithm.”，这个算法源自论文\[2\]。

为了更好的理解回路搜索算法，我也基于论文\[2\]做个实现，放在了github上，参见\[3\]。

\[1\] Josep Llosa, Antonio Gonzalez, Eduard Ayguade, and Mateo Valero. Swing Modulo Scheduling: A Lifetime Sensitive Approach. In Pact96 , pages 80-87, October 1996 (Boston - Massachusetts - USA). \[2\] Donald B. Johnson, Finding all the elementary circuits of a directed graph, SIAM Journal on Computing, 1975. \[3\] [https://github.com/hellogcc/circuit-finding-algorithm](https://github.com/hellogcc/circuit-finding-algorithm)
