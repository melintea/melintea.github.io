---
layout: post
title: A note on barriers costs
---

Here is a set of benchmarks for one Intel CPU and one ARM CPU. While you still need to measure for your specific platform (and application), I think this is a fair generalization:

 - On Intel: avoid std::memory_order_seq_cst and std::memory_order_acq_rel; if you can.
 - On ARM: use std::memory_order_relaxed if you can; everything else is expensive and about the same.


Intel:

    | Threads  | R&W:plain   | R&W:cst      | R:acq/W:rel  |R:cons/W:rel  | R&W:acqrel   | R&W:rlxd
    | -------  | --------    | -------      | -----------  |------------  | -------      | -----------
    | 1        | 33127114    | 137253027    | 14527492     |14390159      | 83419552     | 15516709
    | 2        | 65544331    | 111611757    | 40904954     |29246743      | 118626628    | 51504383
    | 4        | 76579883    | 324397517    | 88833451     |93580268      | 344201518    | 68938834
    | 8        | 136478453   | 674507507    | 156113484    |160151688     | 730754759    | 142440185
    | 16       | 343907409   | 1520126869   | 309321524    |307596361     | 1449977644   | 266249518
    | 32       | 535688807   | 2960453288   | 630654026    |556672636     | 2865121020   | 605446567
    | 64       | 1223356722  | 5768121877   | 1142276229   |1156412571    | 5832937534   | 1261543095
    | 128      | 2539714413  | 11670599124  | 2204149831   |2106698462    | 11955427568  | 2328507084

![_config.yml]({{ site.baseurl }}/images/barriers-intel2.png)
![_config.yml]({{ site.baseurl }}/images/barriers-intel1.png)


ARM:

    | Threads  | R&W:plain   | R&W:cst      | R:acq/W:rel  | R:cons/W:rel  | R&W:rlxd
    | -------  | --------    | -------      | -----------  | ------------  | -----------
    | 1        | 37050747    | 64841628     | 64885685     | 64855760      | 22342727
    | 2        | 83245493    | 853507322    | 1024283791   | 1042490241    | 49977590
    | 4        | 75488640    | 2167635118   | 2298913752   | 2350279620    | 78062916
    | 8        | 149902079   | 4411031159   | 4679633671   | 4687182030    | 157835993
    | 16       | 297785589   | 8860286175   | 9465185940   | 9225091012    | 296109828
    | 32       | 588377972   | 17933268280  | 18381362453  | 18402923175   | 596372276 
    | 64       | 1175653706  | 36222742891  | 36580019179  | 37389743218   | 1192288348
    | 128      | 2342565804  | 73656199471  | 73646965131  | 72336900816   | 2353713656

![_config.yml]({{ site.baseurl }}/images/barriers-arm1.png)

