---
layout: post
title: Just use const string ref as arg 
---

I cannot tell any performance gain for using ```std::string_view``` as an argument of a function call insted of a ```const std::string&```

Some [runs](https://github.com/melintea/lpt-tools/blob/main/src/benchmark/string_view_arg.cpp) are a plain draw, some would show a cold-cache advantage of one over the other that will dissapear on subsequent runs.

```
------------------------------------------------------------------------------------
Benchmark                         Time             CPU   Iterations items_per_second
------------------------------------------------------------------------------------
BM_arg(const std::string&)     2817 ns         2815 ns       247650       727.484M/s
BM_arg(std::string_view)       2822 ns         2822 ns       223897       725.773M/s
BM_arg<std::string>            2958 ns         2958 ns       223866       692.294M/s
BM_arg<std::string_view>       2955 ns         2955 ns       223965       693.055M/s
BM_arg<std::string>            2958 ns         2958 ns       223935       692.354M/s
BM_arg<std::string_view>       2955 ns         2955 ns       224019       693.024M/s
BM_arg<std::string>            2958 ns         2958 ns       224000       692.444M/s
BM_arg<std::string_view>       2957 ns         2957 ns       223942       692.578M/s

```
