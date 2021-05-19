> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [oneapi-src.github.io](https://oneapi-src.github.io/oneDNN/dev_guide_link.html)

> oneDNN includes several header files providing C and C++ APIs for the functionality and one or severa......

oneDNN includes several header files providing C and C++ APIs for the functionality and one or several libraries depending on how oneDNN was built.

oneDNN包含了几个提供C或者C++功能接口的头文件和几个根据oneDNNbuild操作设计的库文件

Header Files
------------

<table><tbody><tr><th>File</th><th>Description</th></tr><tr><td><a href="https://oneapi-src.github.io/oneDNN/oneapi_2dnnl_2dnnl_8h.html" title="C API. ">include/oneapi/dnnl/dnnl.h</a></td><td>C header</td></tr><tr><td><a href="https://oneapi-src.github.io/oneDNN/oneapi_2dnnl_2dnnl_8hpp.html" title="C++ API. ">include/oneapi/dnnl/dnnl.hpp</a></td><td>C++ header</td></tr><tr><td><a href="https://oneapi-src.github.io/oneDNN/oneapi_2dnnl_2dnnl__types_8h.html" title="C API types definitions. ">include/oneapi/dnnl/dnnl_types.h</a></td><td>Auxiliary C header</td></tr><tr><td>include/oneapi/dnnl/dnnl_config.h</td><td>Auxiliary C header</td></tr><tr><td>include/oneapi/dnnl/dnnl_version.h</td><td>C header with version information</td></tr></tbody></table>

Libraries
---------

### Linux

<table><tbody><tr><th>File</th><th>Description</th></tr><tr><td>lib/libdnnl.so</td><td>oneDNN dynamic library</td></tr><tr><td>lib/libdnnl.a</td><td>oneDNN static library (if built with <code>DNNL_LIBRARY_TYPE=STATIC</code>)</td></tr></tbody></table>

### macOS

<table><tbody><tr><th>File</th><th>Description</th></tr><tr><td>lib/libdnnl.dylib</td><td>oneDNN dynamic library</td></tr><tr><td>lib/libdnnl.a</td><td>oneDNN static library (if built with <code>DNNL_LIBRARY_TYPE=STATIC</code>)</td></tr></tbody></table>

### Windows

<table><tbody><tr><th>File</th><th>Description</th></tr><tr><td>bin\dnnl.dll</td><td>oneDNN dynamic library</td></tr><tr><td>lib\dnnl.lib</td><td>oneDNN import or full static library (the latter if built with <code>DNNL_LIBRARY_TYPE=STATIC</code>)</td></tr></tbody></table>

Linking to oneDNN
-----------------

The examples below assume that oneDNN is installed in the directory defined in the `DNNLROOT` environment variable.

以下示例假定oneDNN安装在环境变量DNNLROOT定义的目录中。

### Linux/macOS

g++ -std=c++11 -I${DNNLROOT}/include -L${DNNLROOT}/lib simple_net.cpp -ldnnl

clang++ -std=c++11 -I${DNNLROOT}/include -L${DNNLROOT}/lib simple_net.cpp -ldnnl

icpc -std=c++11 -I${DNNLROOT}/include -L${DNNLROOT}/lib simple_net.cpp -ldnnl

> Note
>
> Applications linked dynamically will resolve the dependencies at runtime. Make sure that the dependencies are available in the standard locations defined by the operating system, in the locations listed in the `LD_LIBRARY_PATH` (Linux) or `DYLD_LIBRARY_PATH` (macOS) environment variable or the `rpath` mechanism.
>
> 动态链接的应用程序将在运行时解决依赖性。 确保依赖项在操作系统定义的标准位置，`LD_LIBRARY_PATH`（Linux）或`DYLD_LIBRARY_PATH`（macOS）环境变量或`rpath`机制中列出的位置中可用。

#### Support for macOS hardened runtime

oneDNN requires the [com.apple.security.cs.allow-jit](https://developer.apple.com/documentation/bundleresources/entitlements/com_apple_security_cs_allow-jit) entitlement when it is integrated with an application that uses the macOS [hardened runtime](https://developer.apple.com/documentation/security/hardened_runtime_entitlements). This requirement comes from the fact that oneDNN generates code on the fly and then executes it.

当整合到macOS强化运行时时候，oneDNN需要得到苹果安全的批准，这是基于oneDNN将会动态生成代码并且运行的事实，（需要确保安全性）

It can be enabled in Xcode or passed to `codesign` like this:

```bash
codesign -s "Your identity" --options runtime --entitlements Entitlements.plist [other options...] /path/to/libdnnl.dylib
```

Example `Entitlements.plist`:

```xml
<?xml [version](https://oneapi-src.github.io/oneDNN/group__dnnl__api__service.html#gaad8292408620d0296f22bdf65afb752d)="1.0" encoding="UTF-8"?>

<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">


<plist version="1.0">

<dict>

<key>com.apple.security.cs.allow-jit</key><true/>

</dict>

</plist>
```



### Windows

To link the application from the command line, set up the `LIB` and `INCLUDE` environment variables to point to the locations of the oneDNN headers and libraries.

要从命令行链接应用程序，请设置LIB和INCLUDE环境变量以指向oneDNN头文件（**LIB**）和库（**INCLUDE**）的位置。

```cmd
icl /I%DNNLROOT%\include /Qstd=c++11 /qopenmp simple_net.cpp %DNNLROOT%\lib\dnnl.lib

cl /I%DNNLROOT%\include simple_net.cpp %DNNLROOT%\lib\dnnl.lib
```



Refer to the [Microsoft Visual Studio documentation](https://docs.microsoft.com/en-us/cpp/build/walkthrough-creating-and-using-a-dynamic-link-library-cpp?view=vs-2017) on linking the application using MSVS solutions.

Note

Applications linked dynamically will resolve the dependencies at runtime. Make sure that the dependencies are available in the standard locations defined by the operating system or in the locations listed in the `PATH` environment variable.