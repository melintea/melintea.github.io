---
layout: post
title: Memory model verification tools
---

- [Promela + mmlib + trio additions](https://brilliantsugar.github.io/posts/how-i-learned-to-stop-worrying-and-love-juggling-c++-atomics/)
  - Promela is architecture-agnostic so this is quite a stretch
- [c11tester-llvm](https://github.com/bdemsky/c11tester-llvm)
  - unmaintained
  - the pass can be compiled against llvm8
    - rumored to compile angainst llvm11 but I could not
    - there is an llvm12 branch; I have not tried it
    - seems to be possible to compile it as an out-of-tree but I have not tried it
    - llvm17 and later has a new pass manager - this would be the end of life for the c11tester pass
- [cdschecker](https://github.com/melintea/cdschecker2)
  - too many false positives (see the shared_mutex tests); prefer relacy
  - useable but not with coroutines
  - original is unmaintained
- relacy
  - to be preferred over other tools
  - still maintained: [dvyukov](https://github.com/dvyukov/relacy) and [ccotter](https://github.com/ccotter/relacy)
  - dated codebase but useable
- [genmc](https://github.com/MPI-SWS/genmc)
  - maintained
  - llvm C11 only
- [cppmem](https://www.modernescpp.com/index.php/cppmem-an-overview/)
- [nitpick](https://www21.in.tum.de/~blanchet/nitpick.html)

