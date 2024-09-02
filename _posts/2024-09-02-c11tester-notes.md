---
layout: post
title: c11tester notes
---

Notes about [c11tester-llvm](https://github.com/bdemsky/c11tester-llvm) and https://brilliantsugar.github.io/posts/how-i-learned-to-stop-worrying-and-love-juggling-c++-atomics/

- (just use relacy even though it does not support atomic flags, coroutines and such)
- the extension can be compiled against llvm8
  - rumored to compile angainst llvm11 but I could not
  - there is an llvm12 branch; I have not tried it
  - seems to be possible to compile it as an out-of-tree but I have not tried it

Recipe:
- get llvm8
- graft the pass in the code
- compile and install
- compile the companion lib using the grafted llvm8
- same with the code being verified

```
git clone --depth 1 --branch llvmorg-8.0.1 https://github.com/llvm/llvm-project.git

cmake \
        -S /home/amelinte2/c11tester/llvm-project/llvm \
	-B /home/amelinte2/c11tester/llvm-project/build \
	-DCMAKE_BUILD_TYPE=Release \
	-DCMAKE_INSTALL_PREFIX=/home/amelinte2/c11tester/llvm-install \
	-DLLVM_PARALLEL_COMPILE_JOBS=1 \
	-DLLVM_PARALLEL_LINK_JOBS=1 \
	-DLLVM_PARALLEL_TABLEGEN_JOBS=1 \
	-DLLVM_ENABLE_PROJECTS=all \
	-DCMAKE_CXX_COMPILER=clang++ \
	-DCMAKE_C_COMPILER=clang \
	-DLLVM_ENABLE_EH=ON \
	-DLLVM_ENABLE_RTTI=ON

# graft c11tester-llvm as CDSPasss

cmake --build   llvm-project/build
cmake --install llvm-project/build

# compile c11tester

# compile code being verified
LD_LIBRARY_PATH=c11tester clang++-8 \
    -Xclang -load -Xclang llvm-8.0.1.src/build/lib/libCDSPass.so -Lc11tester/ \
    -lmodel -Ic11tester/include \
    -g -O3 -rdynamic \
    test_c11tester.cc -o out/test_c11tester

./out/test_c11tester

```

