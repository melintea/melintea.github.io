---
layout: post
title: Lockfree gone wrong 
---

![_config.yml]({{ site.baseurl }}/images/lockfree-wrong1.png)

Lockfree is faster. Usually. Unless you do it wrong. And unless you actually measure it to confirm it is faster. And if you measure, sometimes you are surprised.

Here is a typical way to do it wrong[^1]: dumping data into the atomic anytime you can. Because it is lockfree so why worry. But if you do it right, then yes, lockfree is faster.

[^1]: [https://github.com/melintea/lpt-tools/blob/main/src/chrono/examples/lockfree1.cpp]( https://github.com/melintea/lpt-tools/blob/main/src/chrono/examples/lockfree1.cpp)
