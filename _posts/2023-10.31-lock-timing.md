---
layout: post
title: Some lock timings 
---

A set of measurements for various lock types. Use these as a guide and not as full substitute for mesurements in a given software context & hardware:
- these measurements are only indicative for a different architecture. 
- these measurements are completely useless out of the software context where used (i.e. what is the lock used for; how long is it held; etc)


![_config.yml]({{ site.baseurl }}/images/lock-timing-intel2.png)

With above caveats in mind:
- wait-free is best (duh) but there are probably not many places where it can be used. Very cache-friendly too.
- lock-free is next best (ignoring the above note on software contexts which might make it underperform, say, mutexes)
- the mutex is pretty constant with any contention level once contention reaches 2xCPU threads
- spinlocks: while beating the mutexes in low-contention environments, pthread_spinlock_t lose their advantage as soon as the contention keeps growing over a given threshold. In this particular test, on a 4-CPU machine, the mutex wins if contention goes over 32 threads. They are rather CPU-intensive and cache-coherence-destructive - though YMMV with other hardware flavors. And it is not scaling well with contention. Not at all - the time spent per-thread is basically constant. In my tests, test completion times for spinlocks were human-noticeably slower than mutexes (and everyting else) for high contention. Here is is:

![_config.yml]({{ site.baseurl }}/images/lock-timing-intel1.png)

Another note: Microsoft combined both the spinlock and the mutex into a CRITICAL_SECTION object. It is used in the stdio area and the spin defaults to:

```
#define _CORECRT_SPINCOUNT  4000
```

Without apropriate measurements I should have no opinion but: I am not sure this is still appropriate for nowadays architectures. IMO that number should have been scaled inversely to the number of CPUs; then maybe a contention-dynamic spin value is needed - spin less if more contention. 


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

```

