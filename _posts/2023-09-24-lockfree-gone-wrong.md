---
layout: post
title: Lockfree gone wrong 
---

![_config.yml]({{ site.baseurl }}/images/lockfree-wrong1.png)

Lockfree is faster. Usually. Unless you do it wrong. And unless you 
actually measure it to be faster. 

A typical issue [1]: dumping data into the atomic anytime you can. Because 
it is lockfree so why worry.

[1] [https://github.com/melintea/lpt-tools/blob/main/src/chrono/examples/lockfree1.cpp]( https://github.com/melintea/lpt-tools/blob/main/src/chrono/examples/lockfree1.cpp)
