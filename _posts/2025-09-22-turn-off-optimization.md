---
layout: post
title: Locally turning off optimizations
---


```
int [[gnu::optimize("O0")]] f(...) {...}
// also see gnu::noinline
```

```
#pragma GCC push_options
#pragma GCC optimize("O0")

int f(...) {...}

#pragma GCC pop_options
```

