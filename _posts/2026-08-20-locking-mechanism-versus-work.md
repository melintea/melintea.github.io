---
layout: post
title: Locking mechanism versus contention 
---

[https://www.youtube.com/watch?v=UdKqfQ3a_sY&t=36s](https://www.youtube.com/watch?v=UdKqfQ3a_sY&t=36s)

Fastest/best locking mechanism depends also on the parallel workload to be done:
- low contention: use atomics
  - aviod CAS loops
- high contenction: use spinlocks
  - tune the spinlock for high contention
  - do not put the spinlock and its payload on the same cache line (!)
- read ops on shared vars create high latency for writers

![_config.yml]({{ site.baseurl }}/images/pikus-cppnow-2026.png)

