---
layout: post
title: A much faster map search 
---

I have been tipped how to speed up certain map searches. It can cut the search time in half:
- use ```std::less<>```
- if the key is a string: use ```std::string_view```



```
Benchmark                          Time             CPU   Iterations items_per_second
-------------------------------------------------------------------------------------
std::map<std::string, x>::find(const std::string&)
BM_plain<std::string>      138476037 ns    138432049 ns            5       14.7943k/s

std::map<std::string, x>::find(cconst char*)
BM_plain<const char*>      141114308 ns    141072829 ns            5       14.5173k/s

std::map<std::string, x, std::less<>>::find(const std::string&)
BM_less<std::string>       140909209 ns    140904323 ns            5       14.5347k/s

std::map<std::string, x>, std::less<>::find(const char*)
BM_less<const char*>        78508113 ns     78502318 ns            8       26.0884k/s

std::map<std::string, x>, std::less<>::find(std::string_view)
BM_less<std::string_view>   77613553 ns     77608860 ns            9       26.3887k/s

```

