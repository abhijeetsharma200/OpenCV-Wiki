[![OCV Nightly RISC-V (RVV 0.7.1)](https://github.com/opencv/ci-gha-workflow/actions/workflows/OCV-Nightly-RISCV.yaml/badge.svg?event=schedule)](https://github.com/opencv/ci-gha-workflow/actions/workflows/OCV-Nightly-RISCV.yaml)

![](images/riscv.png)

# RISC-V introduction

RISC-V is an Instruction Set Architecture (ISA) which is growing as an alternative to well-known ISAs such as x86/x86_64 and ARM/AArch64. RISC-V is covered by an open-source license allowing royalty-free usage by hardware and software providers. 

Besides base integer instruction set RISC-V processors can implement various architecture extensions, for example **F** for single-precision floating point support or **M** for integer multiplication and division. There exists an extension for vector operations (aka SIMD - single instruction multiple data) - **V** (RVV), which is beneficial for high-performance computing applications like image processing, machine learning and deep learning. This extension can be leveraged by the OpenCV to achieve significant performance improvement across many algorithms. **V**-extension analogs in other platforms are SSE/AVX for x86_64 and NEON/SVE for ARM/AArch64.

Major difference between the **V** extension and other popular SIMD extensions is non-fixed vector length: while SSE instructions operate on 128-bit registers and AVX2 on 256-bit registers, instructions in the **V** extension can operate on whatever register width is provided by an actual hardware. This kind of SIMD specification is also called Scalable SIMD. Similar approach is used by SVE (Scalable Vector Extension) on ARM platforms.

Further in this document we will focus mainly on RVV extension usage in general and in OpenCV specifically.

Links:
- [Wikipedia / RISC-V](https://en.wikipedia.org/wiki/RISC-V)
- [RISC-V / about](https://riscv.org/about/)
- [RISC-V / Vector Extension spec v1.0](https://github.com/riscv/riscv-v-spec/releases/tag/v1.0)
- [RISC-V / Vector Extension spec v0.7.1](https://github.com/riscv/riscv-v-spec/releases/tag/0.7.1)
- [ARM - What is SVE](https://developer.arm.com/documentation/101726/4-0/Learn-about-the-Scalable-Vector-Extension--SVE-/What-is-the-Scalable-Vector-Extension-)

## RISC-V hardware with vector support

Historically the first **V** specification version implemented in hardware was v0.7.1, it differs from finalized v1.0 in several ways. Currently, there are several hardware platforms implementing vector extension available for purchase:

* ❗ RVV v0.7.1 ❗ MangoPi MQ Pro (https://mangopi.org/mqpro) based on D1 SoC which includes C906 RISC-V core by T-Head (doesn't support 64-bit vector element types)
* ❗ RVV v0.7.1 ❗ LicheePi 4A (https://sipeed.com/licheepi4a) based on TH1520 SoC which includes 4xC910 RISC-V cores by T-Head
* ❗ RVV v1.0 ❗ CanMV K230 (https://www.canaan.io/product/k230) based on K230 SoC which includes C908 RISC-V core by T-Head


## RISC-V software with vector support

In order to use **V** extension one should use Linux system which includes kernel built with RVV support. Often it might be the kernel provided by SoC/core manufacturer or mainline kernel with corresponding patches. To check **V** extension support run the following command and check that `isa` line contains letter `v` after `rv64` base specifier:
```.sh
cat /proc/cpuinfo
```
Example output:
```
...
isa             : rv64imafdcvu
...                         ^
                            |
                          here
```


## Development for RISC-V with vector support

While it is possible to write vectorized code using RVV assembly, C/C++ libraries and applications often use vector intrinsics - set of types and functions built into the compiler, which corresponds to machine instructions.

Usually software for RISC-V is built on regular Linux or Window platforms using cross-compilation process. Cross-compiling toolchains include compiler and other libraries and tools required for development. Following toolchains are known to include intrinsics for RVV extension:

- By the [T-Head](https://github.com/T-head-Semi)
   - xuantie-gnu-toolchain 1.x based on GCC 8.4.0 (???) - uses obsolete intrinsics dialect, supports only vector extension v0.7.1
   - xuantie-gnu-toolchain 2.6.1 based on GCC 10.2.0 (https://github.com/T-head-Semi/xuantie-gnu-toolchain, see "Releases") - uses modern but old intrinsics specification and scalable vectors, supports vector extension v0.7.1
- Mainline compilers
   - GCC 13-14 (https://github.com/riscv-collab/riscv-gnu-toolchain) - uses recent intrinsics specification, supports v1.0 of vector extension
   - LLVM/Clang 16-18 (https://github.com/llvm/llvm-project) - uses recent intsinsics specification, supports v1.0 of vector extension

Links:
- [RISC-V RVV intsinsics specification](https://github.com/riscv-non-isa/rvv-intrinsic-doc) (still not finalized, latest version is v.0.12)


## Emulating RISC-V RVV hardware

In order to test applications without hardware one can use emulation software, for example [QEMU](https://www.qemu.org/). Often QEMU emulator is included in toolchain package or can be built together with compiler. There are two operating modes for QEMU: full system emulation and user-mode emulation. In the first case user has to describe and prepare full virtual system with disks and other periferals, install operating system and then work with it like with standalone machine - boot, install software, interact. In the second case user can straight-away run their RISC-V application and QEMU will proxy system calls to the host OS (Linux) - this mode is the best suitable for an application development and debugging, so we will review it in more details.

QEMU user-mode applications have names like `qemu-arm`, `qemu-aarch64`, `qemu-riscv64` - they do not have word `system`. In order to launch an application using QEMU one should pass their command line to the QEMU program like this:
```.sh
qemu-riscv64 <qemu options> ./my-app <app arguments>
```
Following QEMU options are most important to run RISC-V application:
- `-cpu <model>` - select CPU model and feature, for example QEMU provided by T-Head allows selecting specific core to emulate: `c906` and `c906fdv` (with `f`, `d` and `v` extensions), `c908` and `c908v`, `c910` and `c910v`. Generic RISC-V emulation also allows extension selection: `-cpu rv64,v=true,vext_spec=v1.0` will enable RVV v1.0.
- `-L <path>` - set path where LD interpreter will be rooted. Usually this folder is part of toolchain distribution, so this parameter might look like this `-L <toolchain root>/sysroot`.

Other options might be useful for debugging and fine tuning:
- `-help` - show all options and their descriptions
- `-cpu list` - will show all supported CPU models
- `-E <var>=<value>` - set environment variables
- `-g <port>` - wait for GDB connection on selected port

Links:
- [Wikipedia / QEMU](https://en.wikipedia.org/wiki/QEMU)
- [QEMU / Official site](https://www.qemu.org/)


## Debugging RISC-V applications

Usually most convenient way is to debug user application remotely, because either target system do not have a debugger, or the one it has in packages does not match the compiler used for build (e.g. does not support RVV). Remote debugging process with GDB is as follows:
1. Build your application with debugging information enabled, use `-g -Og` compiler options or `-DCMAKE_BUILD_TYPE=Debug` cmake option in case of OpenCV
2. On remote machine run your application with the gdbserver (port can be chosen arbitrarily, e.g. 1234): 
   ```.sh
   gdbserver :<port> ./my-app <args>
   ```
   Program will start and pause immediately waiting for remote connection.
3. On host machine run your application using the GDB from toolchain: 
   ```.sh
   <toolchain root>/bin/riscv64-unknown-linux-gnu-gdb ./my-app <args>
   ```
   Program will be loaded and GDB will wait for further instructions
4. Setup remote connection on the host machine using GDB command `target remote <address>:<port>`, where _address_ is your remote machine IP address or hostname and _port_ is the same as on step 2
5. Debug application from host as usual, e.g. enter the `continue` command to continue execution until it crash and examine program state afterwards.

Similar procedure can be used with the QEMU emulation - set `-g <port>` option to start server and connect using `taget remote :<port>` on the GDB side.


# RISC-V support in OpenCV

OpenCV support RISC-V platform since 2020 and each year it grows and improves. Major contribution has been made by the [T-Head](https://www.t-head.cn/?lang=en) (平头哥半导体有限公司) (`intrin_rvv071.hpp`) and by the Chinese Academy of Sciences (`intrin_rvv.hpp`, `intrin_rvv_scalable.hpp`).

## Universal intrinsics

Most CPU optimizations in the OpenCV are made using Universal Intrinsics - a set of wrappers over platform-specific SIMD compiler intrinsics. Currently, OpenCV has implementations for SSE/AVX/NEON/RVV/VSX/MSA/LSX/WASM intrinsics. 

Historically Universal Intrinsics has gone through three generations:
1. Fixed size intrinsics: types have indication of element size and count, e.g. `v_int8x16` - vectors with 16 8-bit elements (128-bit registers).
2. Wide intrinsics: types have indication of element size only, element count is selected at compile-time, e.g. `v_int8` - vector with 8-bit `VTraits<v_int8>::nlanes` elements, where `nlanes` can be 16, 32 or 64 depending on vector register size (128, 256, 512 bits).
3. Scalable intrinsics: types have indication of element size, element count is selected at run-time depending on platform where it is executed, e.g. `v_int8` - for 8-bit vectors with `V_Traits<v_int8>::vlanes()` elements.

Scalable intrinsics implementation matches well with RVV sizeless vectors paradigm. During summer 2023, as part of OpenCV Summer of Code project, the library has been refactored by @hanliutong using semi-automated approach to support scalable intrinsics in most areas.


## Universal intrinsics for RISC-V RVV

There are 3 implementations of universal intrinsics for RISC-V RVV in OpenCV (files located in `modules/core/include/opencv2/core/hal`):
1. `intrin_rvv071.hpp` - this implementation uses obsolete intrinsics dialect with fixed vector size (128-bit) and can be built only using T-Head toolchain 1.x. It supports only RVV v0.7.1 and can run on T-Head QEMU (`c906fdv`, `c910v`) or MangoPi MQ Pro or LicheePi 4A boards. It shows good efficiency, but future development of this implementation is not possible due to large difference with modern RVV syntax.
2. `intrin_rvv.hpp` - uses modern RVV intrinsics, but is limited to v0.7.1. Can be built only with T-Head toolchain 2.x. This implementation is inefficient because it uses memory buffers as register substitution to overcome sizeless nature of builtin register types. It serves as intermediate step between fixed size implementation and scalable paradigm. Can be executed on same QEMU CPUs and hardware as previous one. Future development is not considered due to inefficiency.
3. `intrin_rvv_scalable.hpp` - uses latest RVV intrinsics, limited to RVV v1.0. Can be built with recent versions of GCC and LLVM toolchains. First real implementation of Scalable Universal Intriniscs. Can be run on T-Head QEMU (`c908v`), mainline QEMU with RVV 1.0 enabled or CanMV K230 board.


## Building OpenCV with RVV support

We will describe all three build variants, one for each Universal Intrinsics implementation listed above.

We assume the following directory structure:
```
<root>/
  - opencv/ - OpenCV repository
  - opencv_extra/ - OpenCV Extra repository with testdata (not required for build)
  - build/ - build location (empty)
```
Prerequisites are: 
- `cmake`
- `ninja-build` (Makefiles generator can be used as well)
- `python3` (?)
- selected RISC-V toolchain installed somewhere (`TOOLCHAIN_ROOT`)

We will use static builds (`BUILD_SHARED_LIBS` option) for deployment convenience, but dynamic builds can be used as well. We also disable OpenCL during builds (`WITH_OPENCL` option) to avoid loading of an experimental OpenCL runtime during OpenCV test execution and reduce testing time and non-relevant failures, this option is not necessary for regular use.

Main difference between build variants is the `<toolchain>.cmake` file being used and some specific options.

### Build for old RVV 0.7.1 intrinsics (1)

_Use T-Head 1.x toolchain._

Build:
```.sh
cd build
PATH=${TOOLCHAIN_ROOT}/bin:${PATH} \
cmake -GNinja \
   -DCMAKE_BUILD_TYPE=Release \
   -DBUILD_SHARED_LIBS=OFF \
   -DWITH_OPENCL=OFF \
   -DCMAKE_TOOLCHAIN_FILE=../opencv/platforms/linux/riscv64-071-gcc.toolchain.cmake \
   ../opencv
ninja
```
Run OpenCV core test using T-Head QEMU:
```.sh
cd build
OPENCV_TEST_DATA_PATH=../opencv_extra/testdata \
${QEMU_DIR}/bin/qemu-riscv64 \
   -L ${TOOLCHAIN_ROOT}/sysroot \
   -cpu c906fdv \
   ./bin/opencv_test_core
```

### Build for new RVV 0.7.1 intrinsics (2)

_Use T-Head 2.6.1 toolchain._

Build:
```.sh
cd build
PATH=${TOOLCHAIN_ROOT}/bin:${PATH} \
cmake -GNinja \
   -DCMAKE_BUILD_TYPE=Release \
   -DBUILD_SHARED_LIBS=OFF \
   -DWITH_OPENCL=OFF \
   -DCMAKE_TOOLCHAIN_FILE=../opencv/platforms/linux/riscv64-gcc.toolchain.cmake \
   -DRISCV_RVV_SCALABLE=OFF \
   ../opencv
ninja
```
Run OpenCV core test using T-Head QEMU:
```.sh
cd build
OPENCV_TEST_DATA_PATH=../opencv_extra/testdata \
${QEMU_DIR}/bin/qemu-riscv64 \
   -L ${TOOLCHAIN_ROOT}/sysroot \
   -cpu c906fdv \
   ./bin/opencv_test_core
```

### Build for new RVV 1.0 intrinsics (3)

_Use mainline GCC or LLVM toolchains._

Build with GCC:
```.sh
cd build
PATH=${TOOLCHAIN_ROOT}/bin:${PATH} \
cmake -GNinja \
   -DCMAKE_BUILD_TYPE=Release \
   -DBUILD_SHARED_LIBS=OFF \
   -DWITH_OPENCL=OFF \
   -DCMAKE_TOOLCHAIN_FILE=../opencv/platforms/linux/riscv64-gcc.toolchain.cmake \
   -DRISCV_RVV_SCALABLE=ON \
   ../opencv
ninja
```
Build with LLVM (also requires GCC for standard libraries):
```.sh
cd build
cmake -GNinja \
   -DCMAKE_BUILD_TYPE=Release \
   -DBUILD_SHARED_LIBS=OFF \
   -DWITH_OPENCL=OFF \
   -DCMAKE_TOOLCHAIN_FILE=../opencv/platforms/linux/riscv64-clang.toolchain.cmake \
   -DRISCV_CLANG_BUILD_ROOT=${LLVM_TOOLCHAIN_ROOT} \
   -DRISCV_GCC_INSTALL_ROOT=${GCC_TOOLCHAIN_ROOT} \
   -DRISCV_RVV_SCALABLE=ON \
   ../opencv
ninja
```
Run OpenCV core test using regular QEMU:
```.sh
cd build
OPENCV_TEST_DATA_PATH=../opencv_extra/testdata \
${QEMU_DIR}/bin/qemu-riscv64 \
   -L ${TOOLCHAIN_ROOT}/sysroot \
   -cpu rv64,v=true,vext_spec=v1.0 \
   ./bin/opencv_test_core
```

**NOTE:** OpenCV uses flexible CPU feature detection during configuration process, so if the compiler does not support RVV optimizations it will be turned off and build will proceed. To avoid this behavior and fail build process in case when compiler does not support RVV, following two options should be added: `-DCPU_BASELINE=RVV -DCPU_BASELINE_REQUIRE=RVV`

**NOTE:** another option can be used to change default CPU option used for RVV detection: `-DCPU_RVV_FLAGS_ON="-march=rv64gcv1p0"`. It can be useful if your want to enable extra RISC-V features or your compiler requires different feature description syntax.
