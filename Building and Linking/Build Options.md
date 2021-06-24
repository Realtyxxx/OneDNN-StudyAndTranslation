# 安装选项

>  原文地址 [oneapi-src.github.io](https://oneapi-src.github.io/oneDNN/dev_guide_build_options.html)

oneDNN supports the following build-time options.

**oneDNN 支持以下的安装时选项。**

<table><tbody><tr><th>CMake Option</th><th>Supported values (defaults in bold)</th><th>Description</th></tr><tr><td>DNNL_LIBRARY_TYPE</td><td><b>SHARED</b>, STATIC</td><td>Defines the resulting library type</td></tr><tr><td>DNNL_CPU_RUNTIME</td><td><b>OMP</b>, TBB, SEQ, THREADPOOL, DPCPP</td><td>Defines the threading runtime for CPU engines</td></tr><tr><td>DNNL_GPU_RUNTIME</td><td><b>NONE</b>, OCL, DPCPP</td><td>Defines the offload runtime for GPU engines</td></tr><tr><td>DNNL_BUILD_EXAMPLES</td><td><b>ON</b>, OFF</td><td>Controls building the examples</td></tr><tr><td>DNNL_BUILD_TESTS</td><td><b>ON</b>, OFF</td><td>Controls building the tests</td></tr><tr><td>DNNL_ARCH_OPT_FLAGS</td><td><em>compiler flags</em></td><td>Specifies compiler optimization flags (see warning note below)</td></tr><tr><td>DNNL_ENABLE_CONCURRENT_EXEC</td><td>ON, <b>OFF</b></td><td>Disables sharing a common scratchpad between primitives in <a href="https://oneapi-src.github.io/oneDNN/group__dnnl__api__attributes.html#ggac24d40ceea0256c7d6cc3a383a0fa07fad521f765a49c72507257a2620612ee96" title="The library manages the scratchpad allocation according to the policy specified by the DNNL_ENABLE_CO...">dnnl::scratchpad_mode::library</a> mode</td></tr><tr><td>DNNL_ENABLE_JIT_PROFILING</td><td><b>ON</b>, OFF</td><td>Enables <a href="https://oneapi-src.github.io/oneDNN/dev_guide_profilers.html">integration with performance profilers</a></td></tr><tr><td>DNNL_ENABLE_PRIMITIVE_CACHE</td><td><b>ON</b>, OFF</td><td>Enables <a href="https://oneapi-src.github.io/oneDNN/dev_guide_primitive_cache.html">primitive cache</a></td></tr><tr><td>DNNL_ENABLE_MAX_CPU_ISA</td><td><b>ON</b>, OFF</td><td>Enables <a href="https://oneapi-src.github.io/oneDNN/dev_guide_cpu_dispatcher_control.html">CPU dispatcher controls</a></td></tr><tr><td>DNNL_ENABLE_CPU_ISA_HINTS</td><td><b>ON</b>, OFF</td><td>Enables <a href="https://oneapi-src.github.io/oneDNN/dev_guide_cpu_isa_hints.html">CPU ISA hints</a></td></tr><tr><td>DNNL_VERBOSE</td><td><b>ON</b>, OFF</td><td>Enables <a href="https://oneapi-src.github.io/oneDNN/dev_guide_verbose.html">verbose mode</a></td></tr><tr><td>DNNL_AARCH64_USE_ACL</td><td>ON, <b>OFF</b></td><td>Enables integration with Arm Compute Library for AArch64 builds</td></tr><tr><td>DNNL_BLAS_VENDOR</td><td><b>NONE</b>, ARMPL</td><td>Defines an external BLAS library to link to for GEMM-like operations</td></tr><tr><td>DNNL_GPU_VENDOR</td><td><b>INTEL</b>, NVIDIA</td><td>Defines GPU vendor for GPU engines</td></tr></tbody></table>

All other building options or values that can be found in CMake files are intended for development/debug purposes and are subject to change without notice. Please avoid using them.

**在 CMake文件中可以找到的所有其他用于开发/调试目的构建选项或值，如有更改，恕不另行通知。 请避免使用它们。**



Common options/通用选项
--------------

CPU Options/CPU选项
-----------

Intel Architecture Processors and compatible devices are supported by oneDNN CPU engine. The CPU engine is built by default and cannot be disabled at build time.

**oneDNN CPU引擎支持Intel架构处理器以及兼容设备。CPU引擎是默认构建的不能在构建时禁用**

### Targeting Specific Architecture/针对特定体系结构

oneDNN uses JIT code generation to implement most of its functionality and will choose the best code based on detected processor features. However, some oneDNN functionality will still benefit from targeting a specific processor architecture at build time. You can use `DNNL_ARCH_OPT_FLAGS` CMake option for this.

**oneDNN使用JIT(just in time)及时生成代码生成去实施大部分的功能，并且根据探测到的处理器特性选择最佳的代码。然而，一些oneDNN功能仍然受益于在构建时指定特定处理器架构，你可以使用`DNNL_ARCH_OPT_FLAGS`CMake选项来指定。**

For Intel(R) C++ Compilers, the default option is `-xSSE4.1`, which instructs the compiler to generate the code for the processors that support SSE4.1 instructions. This option would not allow you to run the library on older processor architectures

**对于Intel(R) C++编译器，默认选项是 `-xSSE4.1`, 它指示编译器为支持 SSE4.1 指令的处理器生成代码。 此选项不允许您在较旧的处理器架构上运行库。**

For GNU* Compilers and Clang, the default option is `-msse4.1`.

**对于GNU编译器还有Clang，默认选项位`-msse4.1`**

>  Warning
>
> 警告
>
>  While use of `DNNL_ARCH_OPT_FLAGS` option gives better performance, the resulting library can be run only on systems that have instruction set compatible with the target instruction set. Therefore, `ARCH_OPT_FLAGS` should be set to an empty string (`""`) if the resulting library needs to be portable.
>
> **虽然使用`DNNL_ARCH_OPT_FLAGS`选项可以得到很好的性能，但生成的库只能运行在拥有与目标指令集兼容的指令集的系统上运行。因此当生成库需要拥有迁移性时`ARCH_OPT_FLAGS`应当设置为空字符串(``""``)。**

### Runtime CPU dispatcher control/运行时CPU调度控制

oneDNN JIT relies on ISA features obtained from the processor it is being run on. There are situations when it is necessary to control this behavior at run-time to, for example, test SSE4.1 code on an AVX2-capable processor. The `DNNL_ENABLE_MAX_CPU_ISA` build option controls the availability of this feature. See [CPU Dispatcher Control](https://oneapi-src.github.io/oneDNN/dev_guide_cpu_dispatcher_control.html) for more information.

**oneDNN JIT 依赖于从运行它的处理器获得的 指令集系统架构特性。 在某些情况下，需要在运行时控制此行为，例如，在支持 AVX2 的处理器上测试 SSE4.1 代码。 `DNNL_ENABLE_MAX_CPU_ISA`构建选项控制此功能的可用性。 有关详细信息，请参阅 [CPU 调度程序控制](https://oneapi-src.github.io/oneDNN/dev_guide_cpu_dispatcher_control.html)。**

### Runtime CPU ISA hints/运行时CPU指令系统架构提示

For performance reasons, sometimes oneDNN JIT needs to be provided with extra hints so as to prefer or avoid particular CPU ISA feature. For example, one might want to disable Zmm registers usage in order to take advantage of higher clock speed. The `DNNL_ENABLE_CPU_ISA_HINTS` build option makes this feature available at runtime. See [CPU ISA Hints](https://oneapi-src.github.io/oneDNN/dev_guide_cpu_isa_hints.html) for more information.

### Runtimes

CPU engine can use OpenMP, Threading Building Blocks (TBB) or sequential threading runtimes. OpenMP threading is the default build mode. This behavior is controlled by the `DNNL_CPU_RUNTIME` CMake option.

#### OpenMP

oneDNN uses OpenMP runtime library provided by the compiler.

Warning

Because different OpenMP runtimes may not be binary-compatible, it's important to ensure that only one OpenMP runtime is used throughout the application. Having more than one OpenMP runtime linked to an executable may lead to undefined behavior including incorrect results or crashes. However as long as both the library and the application use the same or compatible compilers there would be no conflicts.

#### Threading Building Blocks (TBB)

To build oneDNN with TBB support, set `DNNL_CPU_RUNTIME` to `TBB`:

```cmake
$ cmake -DDNNL_CPU_RUNTIME=TBB ..
```



Optionally, set the `TBBROOT` environmental variable to point to the TBB installation path or pass the path directly to CMake:

```cmake
$ cmake -DDNNL_CPU_RUNTIME=TBB -DTBBROOT=/opt/intel/path/tbb ..
```



oneDNN has functional limitations if built with TBB:

*   Winograd convolution algorithm is not supported for fp32 backward by data and backward by weights propagation.

#### Threadpool

To build oneDNN with support for threadpool threading, set `DNNL_CPU_RUNTIME` to `THREADPOOL`

```cmake
$ cmake -DDNNL_CPU_RUNTIME=THREADPOOL ..
```

The `_DNNL_TEST_THREADPOOL_IMPL` CMake variable controls which of the three threadpool implementations would be used for testing: `STANDALONE`, `TBB`, or `EIGEN`. The latter two require also passing `TBBROOT` or `Eigen3_DIR` paths to CMake. For example:

```cmake
$ cmake -DDNNL_CPU_RUNTIME=THREADPOOL -D_DNNL_TEST_THREADPOOL_IMPL=EIGEN -DEigen3_DIR=/path/to/eigen/share/eigen3/cmake ..
```



Threadpool threading support is experimental and has the same limitations as TBB plus more:

*   As threadpools are attached to streams which are only passed during primitive execution, work decomposition is performed statically at the primitive creation time. At the primitive execution time, the threadpool is responsible for balancing the static decomposition from the previous item across available worker threads.

### AArch64 Options

oneDNN includes experimental support for Arm 64-bit Architecture (AArch64). By default, AArch64 builds will use the reference implementations throughout. The following options enable the use of AArch64 optimised implementations for a limited number of operations, provided by AArch64 libraries.

<table><tbody><tr><th>AArch64 build configuration</th><th>CMake Option</th><th>Environment variables</th><th>Dependencies</th></tr><tr><td>Arm Compute Library based primitives</td><td>DNNL_AARCH64_USE_ACL=ON</td><td>ACL_ROOT_DIR=*Arm Compute Library location*</td><td><a href="https://github.com/ARM-software/ComputeLibrary">Arm Compute Library</a></td></tr><tr><td>Vendor BLAS library support</td><td>DNNL_BLAS_VENDOR=ARMPL</td><td>None</td><td><a href="https://developer.arm.com/tools-and-software/server-and-hpc/downloads/arm-performance-libraries">Arm Performance Libraries</a></td></tr></tbody></table>

#### Arm Compute Library

Arm Compute Library is an open-source library for machine learning applications. The development repository is available from [mlplatform.org](https://review.mlplatform.org/#/admin/projects/ml/ComputeLibrary), and releases are also available on [GitHub](https://github.com/ARM-software/ComputeLibrary). The `DNNL_AARCH64_USE_ACL` CMake option is used to enable Compute Library integration:

```cmake
$ cmake -DDNNL_AARCH64_USE_ACL=ON ..
```



This assumes that the environment variable `ACL_ROOT_DIR` is set to the location of Arm Compute Library, which must be downloaded and built independently of oneDNN.

Warning

For a debug build of oneDNN it is advisable to specify a Compute Library build which has also been built with debug enabled.

oneDNN is only compatible with Compute Library builds v21.02 or later.

#### Vendor BLAS libraries

oneDNN can use a standard BLAS library for GEMM operations. The `DNNL_BLAS_VENDOR` build option controls BLAS library selection, and defaults to `NONE`. For AArch64 builds with GCC, use the [Arm Performance Libraries](https://developer.arm.com/tools-and-software/server-and-hpc/downloads/arm-performance-libraries):

```cmake
$ cmake -DDNNL_BLAS_VENDOR=ARMPL ..
```



Additional options available for development/debug purposes. These options are subject to change without notice, see [`cmake/options.cmake`](https://github.com/oneapi-src/oneDNN/blob/master/cmake/options.cmake) for details.

GPU Options
-----------

Intel Processor Graphics is supported by oneDNN GPU engine. GPU engine is disabled in the default build configuration.

### Runtimes

To enable GPU support you need to specify the GPU runtime by setting `DNNL_GPU_RUNTIME` CMake option. The default value is `"NONE"` which corresponds to no GPU support in the library.

#### OpenCL*

OpenCL runtime requires Intel(R) SDK for OpenCL* applications. You can explicitly specify the path to the SDK using `-DOPENCLROOT` CMake option.

```cmake
$ cmake -DDNNL_GPU_RUNTIME=OCL -DOPENCLROOT=/path/to/opencl/sdk ..
```

