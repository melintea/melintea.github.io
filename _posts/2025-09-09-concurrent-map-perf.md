---
layout: post
title: boost concurrent_flat_map performance
---

How is boost::concurrent_flat_map comparing to an std::unordered_map protected by an readers-writers lock: 
- x86_64:
  - write: concurrent map wins
  - read: unordered map wins by a large margin (!)
- ARM:
  - write: concurrent map wins by a large margin
  - read: concurrent map wins

YMMV with different key or node types, compilers, CPU, boost implementation, etc.


x86_64 4x Intel(R) Pentium(R) Gold G5420 CPU @ 3.80GHz :

```
--------------------------------------------------------------------------------------------------
Benchmark                                             Time             CPU   Iterations       Rate
--------------------------------------------------------------------------------------------------
BM_unordered_map_setup/real_time/threads:1   4210882327 ns   4209667959 ns            1  7.9685M/s
BM_unordered_map_setup/real_time/threads:2    825552049 ns   1639408470 ns            2 20.3224M/s
BM_unordered_map_setup/real_time/threads:4   2528831688 ns   7204019621 ns            4 3.31719M/s
BM_unordered_map_setup/real_time/threads:8   1592650965 ns   4956882616 ns            8 2.63354M/s
BM_unordered_map_setup/real_time/threads:16   823477319 ns   2550017734 ns           16  2.5467M/s
BM_unordered_map_setup/real_time/threads:32   388500283 ns   1222031068 ns           32 2.69904M/s
BM_unordered_map_setup/real_time/threads:64   204282706 ns    640233391 ns           64 2.56648M/s
BM_unordered_map_setup/real_time/threads:128  104458936 ns    339726029 ns          128 2.50954M/s
---------------------------------------------------------------------------------------
Benchmark                                             Time             CPU   Iterations
---------------------------------------------------------------------------------------
BM_unordered_map_setup/real_time/threads:1_BigO 1334829534.16 (1)  2845248361.07 (1)
BM_unordered_map_setup/real_time/threads:1_RMS         99 %            79 %
--------------------------------------------------------------------------------------------------
Benchmark                                             Time             CPU   Iterations       Rate
--------------------------------------------------------------------------------------------------
BM_boost_map_setup/real_time/threads:1       7540980948 ns   7530113262 ns            1 4.44961M/s
BM_boost_map_setup/real_time/threads:2       1800220571 ns   3598579609 ns            2 9.31953M/s
BM_boost_map_setup/real_time/threads:4        697501682 ns   2232794262 ns            4 12.0266M/s
BM_boost_map_setup/real_time/threads:8        362610840 ns   1178238608 ns            8  11.567M/s
BM_boost_map_setup/real_time/threads:16       177621563 ns    591342902 ns           16 11.8069M/s
BM_boost_map_setup/real_time/threads:32        78495805 ns    289551131 ns           32 13.3584M/s
BM_boost_map_setup/real_time/threads:64        39722562 ns    146635486 ns           64 13.1987M/s
BM_boost_map_setup/real_time/threads:128       20347811 ns     71274757 ns          128 12.8832M/s
---------------------------------------------------------------------------------------
Benchmark                                             Time             CPU   Iterations
---------------------------------------------------------------------------------------
BM_boost_map_setup/real_time/threads:1_BigO  1339687722.83 (1)  1954816252.04 (1)
BM_boost_map_setup/real_time/threads:1_RMS          180 %           123 %
--------------------------------------------------------------------------------------------------
Benchmark                                             Time             CPU   Iterations       Rate
--------------------------------------------------------------------------------------------------
BM_unordered_map_read/real_time/threads:1     745030667 ns    745001351 ns            1 45.0377M/s
BM_unordered_map_read/real_time/threads:2     507194397 ns   1013277556 ns            2 33.0785M/s
BM_unordered_map_read/real_time/threads:4     271916982 ns    885814071 ns            4 30.8499M/s
BM_unordered_map_read/real_time/threads:8     129150421 ns    476561056 ns            8 32.4761M/s
BM_unordered_map_read/real_time/threads:16     65858855 ns    268280091 ns           16 31.8431M/s
BM_unordered_map_read/real_time/threads:32     36245630 ns    134242022 ns           32 28.9297M/s
BM_unordered_map_read/real_time/threads:64     18075367 ns     66743245 ns           64 29.0057M/s
BM_unordered_map_read/real_time/threads:128     7862877 ns     31826218 ns          128 33.3395M/s
---------------------------------------------------------------------------------------
Benchmark                                             Time             CPU   Iterations
---------------------------------------------------------------------------------------
BM_unordered_map_read/real_time/threads:1_BigO 222666899.35 (1)  452718201.40 (1)
BM_unordered_map_read/real_time/threads:1_RMS        114 %            80 %
--------------------------------------------------------------------------------------------------
Benchmark                                             Time             CPU   Iterations       Rate
--------------------------------------------------------------------------------------------------
BM_boost_map_read/real_time/threads:1        4937134249 ns   4932412461 ns            1 6.79634M/s
BM_boost_map_read/real_time/threads:2        1673075567 ns   3336223630 ns            2 10.0278M/s
BM_boost_map_read/real_time/threads:4         677673494 ns   2147316299 ns            4 12.3785M/s
BM_boost_map_read/real_time/threads:8         345794497 ns   1147834124 ns            8 12.1295M/s
BM_boost_map_read/real_time/threads:16        167613224 ns    557313324 ns           16 12.5119M/s
BM_boost_map_read/real_time/threads:32         75484537 ns    275597519 ns           32 13.8913M/s
BM_boost_map_read/real_time/threads:64         37624814 ns    137142448 ns           64 13.9346M/s
BM_boost_map_read/real_time/threads:128        17251726 ns     69653722 ns          128 15.1952M/s
---------------------------------------------------------------------------------------
Benchmark                                             Time             CPU   Iterations
---------------------------------------------------------------------------------------
BM_boost_map_read/real_time/threads:1_BigO   991456513.60 (1)  1575436690.86 (1)
BM_boost_map_read/real_time/threads:1_RMS           159 %           105 %

```


ARM 4x Cortex A76 2400 MHz CPU :

```
--------------------------------------------------------------------------------------------------
Benchmark                                             Time             CPU   Iterations       Rate
--------------------------------------------------------------------------------------------------
BM_unordered_map_setup/real_time/threads:1   3016725374 ns   3016053728 ns            1 11.1228M/s
BM_unordered_map_setup/real_time/threads:2   3331414384 ns   6660887573 ns            2 5.03606M/s
BM_unordered_map_setup/real_time/threads:4   2207351983 ns   8397177562 ns            4  3.8003M/s
BM_unordered_map_setup/real_time/threads:8   2.3313e+10 ns   9.3350e+10 ns            8 179.915k/s
BM_unordered_map_setup/real_time/threads:16  1172653256 ns   4952684355 ns           16 1.78838M/s
BM_unordered_map_setup/real_time/threads:32   412033782 ns   1749867694 ns           32 2.54488M/s
BM_unordered_map_setup/real_time/threads:64   198917892 ns    825415138 ns           64  2.6357M/s
BM_unordered_map_setup/real_time/threads:128   93692304 ns    387746362 ns          128 2.79792M/s
---------------------------------------------------------------------------------------
Benchmark                                             Time             CPU   Iterations
---------------------------------------------------------------------------------------
BM_unordered_map_setup/real_time/threads:1_BigO 4218181081.39 (1)  14917436204.58 (1)
BM_unordered_map_setup/real_time/threads:1_RMS        173 %           200 %
--------------------------------------------------------------------------------------------------
Benchmark                                             Time             CPU   Iterations       Rate
--------------------------------------------------------------------------------------------------
BM_boost_map_setup/real_time/threads:1       8215009115 ns   8212398595 ns            1 4.08453M/s
BM_boost_map_setup/real_time/threads:2       1383581734 ns   2766612314 ns            2 12.1259M/s
BM_boost_map_setup/real_time/threads:4        516728879 ns   2059447591 ns            4 16.2341M/s
BM_boost_map_setup/real_time/threads:8        254041548 ns   1028370931 ns            8 16.5103M/s
BM_boost_map_setup/real_time/threads:16       123716605 ns    506971531 ns           16 16.9513M/s
BM_boost_map_setup/real_time/threads:32        62617175 ns    258362564 ns           32 16.7458M/s
BM_boost_map_setup/real_time/threads:64        30407326 ns    130021141 ns           64 17.2422M/s
BM_boost_map_setup/real_time/threads:128       15135653 ns     64880152 ns          128 17.3196M/s
---------------------------------------------------------------------------------------
Benchmark                                             Time             CPU   Iterations
---------------------------------------------------------------------------------------
BM_boost_map_setup/real_time/threads:1_BigO  1325154754.32 (1)  1878383102.48 (1)
BM_boost_map_setup/real_time/threads:1_RMS          199 %           136 %
--------------------------------------------------------------------------------------------------
Benchmark                                             Time             CPU   Iterations       Rate
--------------------------------------------------------------------------------------------------
BM_unordered_map_read/real_time/threads:1     878100440 ns    877989684 ns            1 38.2125M/s
BM_unordered_map_read/real_time/threads:2    2212028469 ns   4423624232 ns            2 7.58454M/s
BM_unordered_map_read/real_time/threads:4     847765090 ns   3390704983 ns            4 9.89497M/s
BM_unordered_map_read/real_time/threads:8     416020982 ns   1803581469 ns            8  10.082M/s
BM_unordered_map_read/real_time/threads:16    207071801 ns    908432746 ns           16 10.1277M/s
BM_unordered_map_read/real_time/threads:32    100599834 ns    454360486 ns           32 10.4232M/s
BM_unordered_map_read/real_time/threads:64     49871475 ns    227464144 ns           64 10.5128M/s
BM_unordered_map_read/real_time/threads:128    25559839 ns    113760770 ns          128 10.2561M/s
---------------------------------------------------------------------------------------
Benchmark                                             Time             CPU   Iterations
---------------------------------------------------------------------------------------
BM_unordered_map_read/real_time/threads:1_BigO 592127241.32 (1)  1524989814.36 (1)
BM_unordered_map_read/real_time/threads:1_RMS        117 %            97 %
--------------------------------------------------------------------------------------------------
Benchmark                                             Time             CPU   Iterations       Rate
--------------------------------------------------------------------------------------------------
BM_boost_map_read/real_time/threads:1        3854316256 ns   3853760170 ns            1 8.70568M/s
BM_boost_map_read/real_time/threads:2        1182144873 ns   2364018241 ns            2 14.1922M/s
BM_boost_map_read/real_time/threads:4         496086575 ns   1970994047 ns            4 16.9096M/s
BM_boost_map_read/real_time/threads:8         237631380 ns    980484098 ns            8 17.6505M/s
BM_boost_map_read/real_time/threads:16        120908670 ns    501530557 ns           16 17.3449M/s
BM_boost_map_read/real_time/threads:32         59436706 ns    251884015 ns           32 17.6419M/s
BM_boost_map_read/real_time/threads:64         29251954 ns    126946313 ns           64 17.9232M/s
BM_boost_map_read/real_time/threads:128        14585072 ns     63824195 ns          128 17.9734M/s
---------------------------------------------------------------------------------------
Benchmark                                             Time             CPU   Iterations
---------------------------------------------------------------------------------------
BM_boost_map_read/real_time/threads:1_BigO   749295185.89 (1)  1264180204.67 (1)
BM_boost_map_read/real_time/threads:1_RMS           164 %           100 %

```
