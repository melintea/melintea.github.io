---
layout: post
title: A note on the performance of move-constructing
---

Everybody likes to move it move it, as King Julien of Madagascar cartoon fame said. But I heard at least twice that move-constructed objects can negatively impact performance (when compared to plain copy-constructed objects that is). There is an extra memory diffusion which does worsen data access. But I have not been shown any hard data nor could I find any.

Here is what I measured on an 4-cpu Intel(R) Pentium(R) Gold G5420 machine with a 64 bytes cache line for a vector of strings when reading that moved data, when compared against reading plain-constructed data[^1]. If positive, move worsened access, if negative it improved it (I wish):

    CPU Counter      min%    max%    mean%  median%  stddev     
    ---------------  ------  ------  -----  -------  -----------
    Instructions     11.25   11.25   11.25  11.25    0.000113846
    CPU cycles       -12.61  253.97  10.58  5.11     37.20      
    L1 cache misses  3.77    216.78  8.22   4.19     30.09      
    L2 cache misses  3.71    327.55  11.06  4.35     45.67      

On that particular arhitecture, for that particular data structure there is a median of 5% data access penalty and this due to the way the CPU cache is being thrashed around. On different architectures that median is certainly different but for that machine I would think that 5% is quite typical for any data structure that has *been* cache-optimized. 

There is a little 5% stone in your access shoes which will slowly sand away the gains made by move-constructing. If you move-construct, access the data a few times and then destroy the object: most likely you keep most of the construction gain (caveat: that is an educated guess - you need to measure both the move-construct gain and the resulting cache penalty to really be sure). But if you move-construct your globals then access them thousands of times until the process dies, then I would wagger you made the performance worse for these globals. For this particular structure under test, IMO moving is not worth it; details in a moment. 

But see that 250+ % max penalty? That is the initial access, always to be incurred. The cache simply builds itself with significantly more efforts. One-time is not too bad. But in my initial tests, I was hitting the max with every access and I can still modify the test to max it - it is a two lines change to start thrashing the cache again. And chance is a real application will be constantly operating with the cache on the wrong foot unless you optimize the whole. The median in the real world would likely be higher than 5%. Much higher and much closer to the max.

How many reads before this move advantage becomes an actual disadvantage? For this particular structure, is it worth moving? I do not think so. The structure is massive but very fragmented. It was move-constructed twice as fast - a gain of 50%+.  As expected.  But one read costs more than the gain[^2]. Assuming a 5% read degradation as detailed above: after about 20 reads you lost all the advantage.  And again: in this test, that first read is degraded 250% and thus it guarantees you lose performance from the get-go. I was surprised but I cannot argue with the data.  

The CPU churns 11% more instructions for reading moved data as the CPU assumes a data locality that is not there in this case. That is, if you access memory location x, the CPU will load a bunch of extra bytes on the assumption that you will next access x+1 and x+2 and etc. For this structure, some of these extra loads are useless - the next byte is not x+1 but somehwere else, far away, and maybe as far as another memory page. So it has to load x+1 again (and the bunch of extra bytes, as above). 

Move-constructing small objects worsens performance. An object with more intrinsic data locality would exhibit better performance after moving; e.g. an object with bigger contigous data members that span multiple cache lines would be more suitable for moving. I still would not be sure moving would beat plain copy-construction, unless I see the mesurements.  

And this is an 4-CPU ARM Cortex A76 with a 512kb per-core L2 cache & 2Mb L3; 256 bytes cache line (likely a Raspberry 5 documentation typo; my CPU checks report a 64 bytes line)[^1]:

    CPU Counter           min%    max%    mean%  median%  stddev     
    ---------------       ------  ------  -----  -------  -----------
    Instructions          11.2448 11.2449 11.2449 11.2449 1.35198e-05 
    CPU cycles            4.53292 314.231 11.4682 5.27542 43.6922 
    L1 cache misses       14.0285 1648.48 46.7225 16.0358 231.146 
    L2 cache misses       36.3684 9995.55 239.577 53.1118 1407.86 
    Branch mispredictions 26.4151 136.364 67.1721 62.9479 24.8802

Here the measured move gain was a paltry 1.5%[^2]. Against this there is a median of 50% access performance loss and a first-time cold-cache one that is 16x worse if move-constructing. I do not see how moving can be an improvement for any structure on that CPU; and I suspect ARMs at large behave the same.

[^1]: [https://github.com/melintea/lpt-tools/blob/main/src/papi/examples/papimove3.cpp](https://github.com/melintea/lpt-tools/blob/main/src/papi/examples/papimove3.cpp)
[^2]: [https://github.com/melintea/lpt-tools/blob/main/src/papi/examples/papimove4.cpp](https://github.com/melintea/lpt-tools/blob/main/src/papi/examples/papimove4.cpp)

