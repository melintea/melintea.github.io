---
layout: post
title: Half or double 
---

The performance lottery.

I have been tipped off how to speed up map searches when the key is a string: use transparent comparators. 

For a map, it can cut the search time almost in half:
- use ```std::less<>```
- use ```std::string_view```

But, surprise: ```std::unordered_map``` exhibits a significant **loss** of search performance 
when fitted with transparent comparators. Unless I am doing it wrong...

To start, note how the baseline hash performance is **much** worse than the baseline map one.
This goes agains the usual wisdom of hashes being faster than trees for large datasets. 
The string keys are quite long and the hash time dwarfs both the plain string comparison time
and the tree search time.

But the important note is: the transparent comparators are doubling the baseline search time...


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


std::unordered_map<std::string, x>::find(const std::string&)
BM_Uplain<std::string>      406852133 ns    406840611 ns            2       5.03391k/s

std::unordered_map<std::string, x>::find(cconst char*)
BM_Uplain<const char*>      424539962 ns    424523349 ns            2       4.82423k/s

std::unordered_map<std::string, x, std::equal_to<>>::find(cconst char*)
BM_Uless<const char*>       668691575 ns    668647711 ns            1        3.0629k/s

std::unordered_map<std::string, x, std::equal_to<>>::find(const std::string&)
BM_Uless<std::string>       695774248 ns    695744144 ns            1       2.94361k/s

std::munordered_mapap<std::string, x, std::equal_to<>>, std::less<>::find(std::string_view)
BM_Uless<std::string_view>  631857955 ns    631816680 ns            1       3.24145k/s
```
