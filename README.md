# NesCheck on LLVM 10.0.0
For information of NesCheck, see the paper in [AsiaCCS 2017](https://hexhive.epfl.ch/publications/files/17AsiaCCS2.pdf)

To put is simple, NesCheck analyzes and classifies pointers in terms of memory safety related operations (e.g., pointer arithmetic, type cast), the classification scheme is similar to [CCured](https://people.eecs.berkeley.edu/~necula/Papers/ccured_popl02.pdf).

Since the original NesCheck framework is designed for running on TinyOS and analyzing programs of embedded devices (written in nesC, a C dialect), to make it applicable to C and C++ programs and support Linux, we upgrade NesCheck pass to LLVM 10.0 on Ubuntu 20.04 with Linux kernel version 5.8.0-48-generic.

## Installation
Make sure your base system is Ubuntu 20.04, this tool hasn’t been tested on other platforms up to now, so Ubuntu 20.04 is the preferred choice.

### Build LLVM 10.0.0
Download LLVM 10.0.0 source code package from the link: https://github.com/llvm/llvm-project/tree/llvmorg-10.0.0

Create a build directory for building LLVM 10.0, make sure ```build``` is outside ```llvm``` directory since LLVM doesn't support in-tree build.
```bash 
cd llvm-project-llvmorg-10.0.0
mkdir build
```

Enter the build directory and perform cmake:
```bash 
cd build
cmake -DLLVM_ENABLE_PROJECTS="clang;libcxx;libcxxabi" -G "Unix Makefiles" ../llvm

```
If everything goes well, you should be able to see a ```Makefile``` in ```build``` run ```make``` to build LLVM(takes time).
```bash 
make
```
### Build SVF
Download SVF source code package from the link below: https://github.com/SVF-tools/SVF/tree/f683d8f21416e9b80baa18ad9d8037a9ed3897f3


Set the environment variable for building SVF:
```bash
export LLVM_DIR='llvm-project-llvmorg-10.0.0/build/bin' 
```
Build SVF using ```build.sh```
```bash
cd SVF
source ./build.sh
```

### Build NesCheck

Copy ```NesCheck``` directory in the repo into ```llvm-project-llvmorg-10.0.0/llvm/lib/Transforms``` directory.

Modify the ```CMakeLists.txt``` in the ```NesCheck``` directory, adjust the path of ```include_directories``` and ```target_link_libraries```.

Modify the ```CMakeLists.txt``` in the ```llvm-project-llvmorg-10.0.0/llvm``` directory, add ```add_subdirectory(lib/Transforms/NesCheck)``` at line 897.

Enter the build directory and perform cmake again:
```bash 
cd llvm-project-llvmorg-10.0.0/build
cmake -DLLVM_ENABLE_PROJECTS="clang;libcxx;libcxxabi" -G "Unix Makefiles" ../llvm
```

Build NesCheck pass:
```bash 
make NesCheck
```

## Usage
Generate bitcode for ```neschecklib.c``` in the repo:
```bash
./llvm-project-llvmorg-10.0.0/build/bin/clang-10 -O0 -g -emit-llvm neschecklib.c -c -o neschecklib.bc
```

Link ```neschecklib.bc``` with the bitcode you want to analyze ```target.bc```:
```bash
./llvm-project-llvmorg-10.0.0/build/bin/llvm-link neschecklib.bc target.bc -o target.linked.bc
```

In case you need help with building whole-program (or whole-library) LLVM bitcode files from an unmodified C or C++ source package, please see [wllvm](https://github.com/travitch/whole-program-llvm).

It's time to run the analysis:
```bash
./llvm-project-llvmorg-10.0.0/build/bin/opt -o target.opt.bc -load llvm-project-llvmorg-10.0.0/build/lib/libNesCheck.so -nescheck -stats -time-passes < target.linked.bc > target.nescheckout
```

