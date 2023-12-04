---
layout: post
title: Some lock timings 
---

Made a set of measurements for various lock types. These can be used as a guide and not as full substitute for mesurements in a given software context & hardware:
- these measurements are only indicative for a different architecture. 
- these measurements are even less useful out of the software context where used (i.e. what is the lock used for; how long is it held; etc)


![_config.yml]({{ site.baseurl }}/images/lock-timing-intel2.png)
![_config.yml]({{ site.baseurl }}/images/lock-timing-arm1.png)

With above caveats in mind, I think these are fair inferences from the data:
- wait-free is best (duh) but there are probably not many places where it can be used. Very cache-friendly too.
- lock-free is next best (but keep in mind the above note on software contexts which might make it underperform, say, mutexes; see e.g. {% link _posts/2023-09-24-lockfree-gone-wrong.md %}[^1])
- the std::mutex is pretty constant with any contention level once contention reaches 2xCPU threads (Intel); or more than once CPU (ARM)
- pthread_spinlock_t: 
    - They are rather CPU-intensive and cache-coherence-destructive - though YMMV with other hardware flavors. And it is not scaling well with contention. Not at all - the time spent per-thread is basically constant. In my tests, test completion times for spinlocks were human-noticeably slower than mutexes (and everyting else) for high contention.
    - ARM: just avoid it. It loses  any edge over the mutex at contention levels above 3 on a 4-CPU machine. 
    - Intel: while beating the std::mutex in low-contention environments, pthread_spinlock_t lose their advantage as soon as the contention keeps growing over a given threshold. In this particular test, on a 4-CPU Intel machine, the mutex wins if contention goes over 32 threads. 
    - Custom-written spinlocks could behave better: Fedor Pikus's one has very good performance[^2]. Or the Rigtorp's one[^3]. It is not a simple task[^4] and IMO the improved performance stems from periodically yielding/sleeping (cheating a bit?) but I have no measurements yet; with the sleep algorithm quite dependent on the architecture. 
    - Here is the Intel damage:

![_config.yml]({{ site.baseurl }}/images/lock-timing-intel1.png)

Another note: Microsoft must have run similar tests, on Intel, years ago. It combined both the spinlock and the mutex into a CRITICAL_SECTION object. It is now used in the stdio area and the spin defaults to:

```
#define _CORECRT_SPINCOUNT  4000
```

So it spins 4000 times before deciding to sleep on the mutex. Without apropriate measurements I should have no opinion but: I am not sure this is still appropriate for nowadays architectures. IMO that fixed spin number should have been scaled inversely to the number of CPUs; then maybe a contention-dynamic spin value is needed - spin less with increased contention. 

As for ARM, there does not seem to be much to gain with a CRITICAL_SECTION.


<!--
```

Run on (4 X 3800 MHz CPU s)
CPU Caches:
  L1 Data 32 KiB (x2)
  L1 Instruction 32 KiB (x2)
  L2 Unified 256 KiB (x2)
  L3 Unified 4096 KiB (x1)
Load Average: 26.31, 9.47, 4.09
***WARNING*** CPU scaling is enabled, the benchmark real time measurements may be noisy and will incur extra overhead.
---------------------------------------------------------------------------------------
Benchmark                                  Time             CPU   Iterations       Rate
---------------------------------------------------------------------------------------
BM_Mutex/real_time/threads:1       372183265 ns    371930734 ns            2 45.0778M/s
BM_Mutex/real_time/threads:2       828728313 ns   1642745438 ns            2 20.2445M/s
BM_Mutex/real_time/threads:4       665142040 ns   1880502575 ns            4 12.6118M/s
BM_Mutex/real_time/threads:8       364317837 ns   1098455780 ns            8 11.5128M/s
BM_Mutex/real_time/threads:16      173021583 ns    549231827 ns           16 12.1208M/s
BM_Mutex/real_time/threads:32       92571975 ns    304178543 ns           32 11.3271M/s
BM_Mutex/real_time/threads:64       42670633 ns    142381319 ns           64 12.2869M/s
BM_Mutex/real_time/threads:128      22902619 ns     81923167 ns          128  11.446M/s
----------------------------------------------------------------------------
Benchmark                                  Time             CPU   Iterations
----------------------------------------------------------------------------
BM_Mutex/real_time/threads:1_BigO 320192283.19 (1)  758918673.04 (1)
BM_Mutex/real_time/threads:1_RMS          87 %            86 %
---------------------------------------------------------------------------------------
Benchmark                                  Time             CPU   Iterations       Rate
---------------------------------------------------------------------------------------
BM_PtSpin/real_time/threads:1      243797350 ns    243799186 ns            3 45.8775M/s
BM_PtSpin/real_time/threads:2      136664955 ns    272271103 ns            6 40.9206M/s
BM_PtSpin/real_time/threads:4      129163524 ns    391785787 ns            4 64.9456M/s
BM_PtSpin/real_time/threads:8      151475798 ns    499771150 ns            8 27.6896M/s
BM_PtSpin/real_time/threads:16     118245510 ns    407959808 ns           16 17.7356M/s
BM_PtSpin/real_time/threads:32      99280671 ns    358014631 ns           32 10.5617M/s
BM_PtSpin/real_time/threads:64      98019994 ns    365460396 ns           64 5.34879M/s
BM_PtSpin/real_time/threads:128     59609581 ns    242166590 ns          128 4.39768M/s
----------------------------------------------------------------------------
Benchmark                                  Time             CPU   Iterations
----------------------------------------------------------------------------
BM_PtSpin/real_time/threads:1_BigO 129532172.94 (1)  347653581.55 (1)
BM_PtSpin/real_time/threads:1_RMS         39 %            24 %
---------------------------------------------------------------------------------------
Benchmark                                  Time             CPU   Iterations       Rate
---------------------------------------------------------------------------------------
BM_LockFree/real_time/threads:1    279137910 ns    279135775 ns            3 40.0691M/s
BM_LockFree/real_time/threads:2    180197248 ns    359024386 ns            4 46.5524M/s
BM_LockFree/real_time/threads:4    116204665 ns    386142218 ns            8 36.0941M/s
BM_LockFree/real_time/threads:8     68312912 ns    229082265 ns            8 61.3984M/s
BM_LockFree/real_time/threads:16    35720324 ns    133002928 ns           16 58.7103M/s
BM_LockFree/real_time/threads:32    17453380 ns     62109020 ns           64 30.0393M/s
BM_LockFree/real_time/threads:64     8257981 ns     32608820 ns          128 31.7443M/s
BM_LockFree/real_time/threads:128    2753002 ns     13097931 ns          256 47.6106M/s
----------------------------------------------------------------------------
Benchmark                                  Time             CPU   Iterations
----------------------------------------------------------------------------
BM_LockFree/real_time/threads:1_BigO 88504677.73 (1)  186775417.82 (1)
BM_LockFree/real_time/threads:1_RMS        104 %            74 %
---------------------------------------------------------------------------------------
Benchmark                                  Time             CPU   Iterations       Rate
---------------------------------------------------------------------------------------
BM_WaitFree/real_time/threads:1    213561674 ns    213344938 ns            3 52.3727M/s
BM_WaitFree/real_time/threads:2     92615983 ns    184035918 ns            8  45.287M/s
BM_WaitFree/real_time/threads:4     47936493 ns    148999780 ns           12 58.3314M/s
BM_WaitFree/real_time/threads:8     31274736 ns    107253557 ns           32 33.5279M/s
BM_WaitFree/real_time/threads:16    13111093 ns     55743115 ns           64 39.9881M/s
BM_WaitFree/real_time/threads:32     6433701 ns     27816904 ns          128 40.7454M/s
BM_WaitFree/real_time/threads:64     3254567 ns     14226486 ns          320 32.2186M/s
BM_WaitFree/real_time/threads:128    1856911 ns      6830837 ns         1792 10.0837M/s
----------------------------------------------------------------------------
Benchmark                                  Time             CPU   Iterations
----------------------------------------------------------------------------
BM_WaitFree/real_time/threads:1_BigO 51255644.76 (1)  94781441.93 (1)
BM_WaitFree/real_time/threads:1_RMS        132 %            79 %


Run on (4 X ARM Cortex A76 2400 MHz CPU s)
Load Average: 0.62, 1.01, 2.31
***WARNING*** CPU scaling is enabled, the benchmark real time measurements may be noisy and will incur extra overhead.
---------------------------------------------------------------------------------------
Benchmark                                  Time             CPU   Iterations       Rate
---------------------------------------------------------------------------------------
BM_Mutex/real_time/threads:1       391728653 ns    391664156 ns            2 42.8287M/s
BM_Mutex/real_time/threads:2      1566944408 ns   3133732396 ns            2  10.707M/s
BM_Mutex/real_time/threads:4       612992238 ns   2392159907 ns            4 13.6847M/s
BM_Mutex/real_time/threads:8       307948406 ns   1236959937 ns            8 13.6202M/s
BM_Mutex/real_time/threads:16      152801352 ns    616390236 ns           16 13.7247M/s
BM_Mutex/real_time/threads:32       76441163 ns    309636400 ns           32 13.7174M/s
BM_Mutex/real_time/threads:64       37117348 ns    154299111 ns           64 14.1251M/s
BM_Mutex/real_time/threads:128      18554308 ns     76519239 ns          128 14.1285M/s
----------------------------------------------------------------------------
Benchmark                                  Time             CPU   Iterations
----------------------------------------------------------------------------
BM_Mutex/real_time/threads:1_BigO 395565984.59 (1)  1038920172.78 (1)
BM_Mutex/real_time/threads:1_RMS         122 %           103 %
---------------------------------------------------------------------------------------
Benchmark                                  Time             CPU   Iterations       Rate
---------------------------------------------------------------------------------------
BM_PtSpin/real_time/threads:1      335765098 ns    335708915 ns            2 49.9671M/s
BM_PtSpin/real_time/threads:2      962264267 ns   1924431122 ns            2 17.4351M/s
BM_PtSpin/real_time/threads:4      783641221 ns   3133445422 ns            4 10.7047M/s
BM_PtSpin/real_time/threads:8      817023173 ns   3622510467 ns            8 5.13364M/s
BM_PtSpin/real_time/threads:16     566111544 ns   2988853489 ns           16 3.70449M/s
BM_PtSpin/real_time/threads:32     555493463 ns   2963675116 ns           32 1.88765M/s
BM_PtSpin/real_time/threads:64     495072239 ns   2675959007 ns           64 1.05901M/s
BM_PtSpin/real_time/threads:128    443209706 ns   2504237100 ns          128 591.467k/s
----------------------------------------------------------------------------
Benchmark                                  Time             CPU   Iterations
----------------------------------------------------------------------------
BM_PtSpin/real_time/threads:1_BigO 619822588.94 (1)  2518602579.64 (1)
BM_PtSpin/real_time/threads:1_RMS         32 %            38 %
---------------------------------------------------------------------------------------
Benchmark                                  Time             CPU   Iterations       Rate
---------------------------------------------------------------------------------------
BM_LockFree/real_time/threads:1    304616840 ns    304599629 ns            2 55.0765M/s
BM_LockFree/real_time/threads:2   1293569432 ns   2586995454 ns            2 12.9697M/s
BM_LockFree/real_time/threads:4    432387447 ns   1725480652 ns            4 19.4007M/s
BM_LockFree/real_time/threads:8    206437024 ns    885006719 ns            8 20.3176M/s
BM_LockFree/real_time/threads:16   101065450 ns    446462100 ns           16 20.7504M/s
BM_LockFree/real_time/threads:32    49824142 ns    222821562 ns           32 21.0455M/s
BM_LockFree/real_time/threads:64    24972667 ns    111576392 ns           64 20.9945M/s
BM_LockFree/real_time/threads:128   12051590 ns     55933942 ns          128 21.7518M/s
----------------------------------------------------------------------------
Benchmark                                  Time             CPU   Iterations
----------------------------------------------------------------------------
BM_LockFree/real_time/threads:1_BigO 303115573.90 (1)  792359556.14 (1)
BM_LockFree/real_time/threads:1_RMS        132 %           107 %
---------------------------------------------------------------------------------------
Benchmark                                  Time             CPU   Iterations       Rate
---------------------------------------------------------------------------------------
BM_WaitFree/real_time/threads:1    195901983 ns    195836796 ns            4 42.8204M/s
BM_WaitFree/real_time/threads:2    476774620 ns    953498468 ns            2  35.189M/s
BM_WaitFree/real_time/threads:4    235439963 ns    936727617 ns            4 35.6295M/s
BM_WaitFree/real_time/threads:8    113586573 ns    468203491 ns            8 36.9261M/s
BM_WaitFree/real_time/threads:16    54961626 ns    235692911 ns           16 38.1567M/s
BM_WaitFree/real_time/threads:32    27041085 ns    118543902 ns           32 38.7771M/s
BM_WaitFree/real_time/threads:64    13129261 ns     58700909 ns           64 39.9328M/s
BM_WaitFree/real_time/threads:128    5941350 ns     29392164 ns          128  44.122M/s
----------------------------------------------------------------------------
Benchmark                                  Time             CPU   Iterations
----------------------------------------------------------------------------
BM_WaitFree/real_time/threads:1_BigO 140347057.76 (1)  374574532.35 (1)
BM_WaitFree/real_time/threads:1_RMS        107 %            94 %

```
-->

[^1]: [https://melintea.github.io/lockfree-gone-wrong/](https://melintea.github.io/lockfree-gone-wrong/)
[^2]: [https://github.com/melintea/lpt-tools/blob/main/include/lpt/spinlock.hpp](https://github.com/melintea/lpt-tools/blob/main/include/lpt/spinlock.hpp)
[^3]: [https://coffeebeforearch.github.io/2020/11/07/spinlocks-7.html](https://coffeebeforearch.github.io/2020/11/07/spinlocks-7.html)
[^4]: [https://rigtorp.se/spinlock/](https://rigtorp.se/spinlock/)

