---
layout: post
title: A note on the performance of move-constructing
---

I heard at least twice that move-constructed objects can negatively impact performance (when compared to plain copy-constructed objects that is). There is an extra memory diffusion which does worsen data access. But I have not been shown any hard data nor could I find any.

Here is what I measured on an 4-cpu Intel(R) Pentium(R) Gold G5420 machine for a vector of strings when reading that data [1]:

    CPU Counter     | min%   | max%   | mean% | median% | stddev     
    --------------- | ------ | ------ | ----- | ------- | -----------
    Instructions    | 11.25  | 11.25  | 11.25 | 11.25   | 0.000113846
    CPU cycles      | -12.61 | 253.97 | 10.58 | 5.11    | 37.20      
    L1 cache misses | 3.77   | 216.78 | 8.22  | 4.19    | 30.09      
    L2 cache misses | 3.71   | 327.55 | 11.06 | 4.35    | 45.67      

On that particular arhitecture, for that particular data structure there is a median of 5% data access penalty and this due to the way the CPU cache is being thrashed around. On different architectures that median is certainly different but for that machine I would think that 5% is quite typical for any data structure that has not been cache-optimized. 

So that little 5% stone in your access shoes will slowly sand away the gains made by move-constructing. If you move-construct, access the data a few times and then destroy the object: most likely you keep most of the construction gain (caveat: that is an educated guess - you need to measure to really be sure). But if you move-construct your globals then access them thousands of times until the process dies, then I would wagger you made the performance worse for these globals.

And see that 250+ % max penalty? That is the initial access, always to be incurred. The cache simply builds itself with significantly more efforts. One-time is not too bad. But in my initial tests, I was hitting the max with every access and I can still modify the test to max it - it is a two lines change that thrashes the cache. And a real application might do just that - constantly operating with the cache on the wrong foot unless you optimize the whole binary. 

PS: I am still at a loss why we have 11% more instructions for reading moved data. Very likely it is cache related (duh).

[1] [https://github.com/melintea/lpt-tools/blob/main/src/papi/examples/papimove3.cpp](https://github.com/melintea/lpt-tools/blob/main/src/papi/examples/papimove3.cpp)

