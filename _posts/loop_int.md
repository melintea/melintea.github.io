---
layout: post
title: Use int as a loop variable
---

A little gem, courtesy of Fedor Pikus & team: use int as a loop variable. Even though, most of the time, using an unsigned variable is the logic thing. There is some performance that can be extracted by the signed variable. For x86, the case is clear (YMMV with another CPU or compiler). For ARM, there is still some little performance that can be squeezed (YMMV, etc.). Though the difference here is within the range of statistical noise, it was always tilted in favor of the int in all the runs I made.

The compiler generates different code for the increment (duh): the unsigned increment has to wrap around on overflow and the compiler adds checks for it; whereas the signed increment overflow is UB and and the compiler is free to skip the overflow check.


```
x86
-----------------------------------------------------------------------------------------
Benchmark                              Time             CPU   Iterations items_per_second
-----------------------------------------------------------------------------------------
BM_loop<int>/1048576              345951 ns       345936 ns         1997       3.03113G/s
BM_loop<unsigned int>/1048576     564400 ns       564391 ns         1172       1.85789G/s
BM_loop<size_t>/1048576           561152 ns       561144 ns         1261       1.86864G/s

ARM
-----------------------------------------------------------------------------------------
Benchmark                              Time             CPU   Iterations items_per_second
-----------------------------------------------------------------------------------------
BM_loop<int>/1048576              892288 ns       892206 ns          784       1.17526G/s
BM_loop<unsigned int>/1048576     896298 ns       896066 ns          781        1.1702G/s
BM_loop<size_t>/1048576           895692 ns       895624 ns          781       1.17078G/s

```


