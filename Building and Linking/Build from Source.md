>  原文地址 [oneapi-src.github.io](https://oneapi-src.github.io/oneDNN/dev_guide_build.html)

> Download oneDNN source code or clone the repository

# 从源安装

## Download the Source Code
------------------------

Download [oneDNN source code](https://github.com/oneapi-src/oneDNN/archive/master.zip) or clone [the repository](https://github.com/oneapi-src/oneDNN.git).

## Build the Library
-----------------

**保证所有的软件依赖项都到位，并且最少达到受支持的最低版本**

**oneDNN的build系统是基于CMake的**

* **`CMAKE_INSTALL_PREFIX`控制库文件安装位置**

* **`CMAKE_BUILD_TYPE` 选择安装版本 (`Release`, `Debug`, `RelWithDebInfo`).**

  - **Release —— 不可以打断点调试，程序开发完成后发行使用的版本，占的体积小。 它对代码做了优化，因此速度会非常快，**
    **在编译器中使用命令： `-O3 -DNDEBUG` 可选择此版本。**
  - **Debug ——调试的版本，体积大。**
    **在编译器中使用命令： `-g` 可选择此版本。**
  - **MinSizeRel—— 最小体积版本**
    **在编译器中使用命令：`-Os -DNDEBUG`可选择此版本。**
  - **RelWithDebInfo—— 既优化又能调试。**
    **在编译器中使用命令：`-O2 -g -DNDEBUG`可选择此版本。**

* **`CMAKE_PREFIX_PATH` 如果你的依赖项没有安装在标准默认位置，需要用这个参数指定需要查找搜索的目录**

  See [Build Options](./Build Options.md) for detailed description of build-time configuration options.

> **CPU选项，指定特殊架构运用特定优化指令集，OpenMP。线程创建块，arm架构选项，ACL库，BLAS库，**
>
> **GPU选项，运行时（指定可用），OpenCL**

### Linux/macOS

#### GCC, Clang, or Intel C/C++ Compiler

* Set up the environment for the compiler

  **给编译器设置好环境**

*   Configure CMake and generate makefiles
    
    **配置configure CMake，生成makefiles**
    
    ```bash
    mkdir -p build
    
    cd build
    
    # Uncomment the following lines to build with Clang
    
    # export CC=clang
    
    # export CXX=clang++
    
    # Uncomment the following lines to build with Intel C/C++ Compiler
    
    # export CC=icc
    
    # export CXX=icpc
    
    cmake .. <extra build options>
    ```
    
* Build the library

  **创建库**

  ```bash
  make -j
  ```

  

#### oneAPI DPC++ Compiler

*   Set up the environment for oneAPI DPC++ Compiler. For Intel oneAPI Base Toolkit distribution installed to default location you can do this using `setenv.sh` script
    
    **为 oneAPI编译器配置环境，如果是安装到默认位置的发行版，可以直接通过以下命令**
    
    ```bash
    source /opt/intel/oneapi/setvars.sh
    ```
    
    
    
*   Configure CMake and generate makefiles
    
    ```bash
    mkdir -p build
    
    cd build
    
    export CC=clang
    
    export CXX=clang++
    
    cmake .. \
    
    -DDNNL_CPU_RUNTIME=DPCPP
    
    -DDNNL_GPU_RUNTIME=DPCPP
    
    <extra build options>
    ```
    
    
    
* Build the library

  **建库**

  ```bash
  make -j
  ```

  

#### GCC targeting AArch64 on x64 host

**针对64位主机上的arm64位架构**

* Set up the environment for the compiler

  **给编译器搭建好环境**

*   Configure CMake and generate makefiles
    
    ```bash
    export CC=aarch64-linux-gnu-gcc
    
    export CXX=aarch64-linux-gnu-g++
    
    cmake .. \
    
    -DCMAKE_SYSTEM_NAME=Linux \
    
    -DCMAKE_SYSTEM_PROCESSOR=AARCH64 \
    
    -DCMAKE_LIBRARY_PATH=/usr/aarch64-linux-gnu/lib \
    
    <extra build options>
    ```
    
    
    
* Build the library

  ```bash
  make -j
  ```

  

#### GCC with Arm Compute Library (ACL) on AArch64 host

**在arm64位架构主机上使用ACL库gcc**

* Set up the environment for the compiler

  **建造环境**

*   Configure CMake and generate makefiles
    
    ```bash
    export ACL_ROOT_DIR=<path/to/Compute Library>
    
    cmake .. \
    
    	-DDNNL_AARCH64_USE_ACL=ON \
    
      <extra build options>
    ```
    
*   Build the library

    ```bash
    make -j
    ```

    

### Windows

#### Microsoft Visual C++ Compiler or Intel C/C++ Compiler

**微软或者intel的 c/c++编译器**

*   Generate a Microsoft Visual Studio solution
    
    **生成Microsoft Visual Studio解决方案**
    
    ```bash
    mkdir build
    
    cd build
    
    cmake -G "Visual Studio 15 2017 Win64" ..
    ```
    
    For the solution to use the Intel C++ Compiler, select the corresponding toolchain using the cmake `-T` switch:
    
    **如果解决方案使用intel 的C++编译器，那么使用cmake `-T `选择相应的工具链**
    
    ```bash
    cmake -G "Visual Studio 15 2017 Win64" -T "Intel C++ Compiler 19.0" ..
    ```
    
* Build the library

  ```bash
  make --build
  ```

  

> Note
>
> You can also open `oneDNN.sln` to build the project from the Microsoft Visual Studio IDE.
>
> **也可以打开oneDNN.sln文件来给Microsoft Visual Studio IDE建立project**

#### oneAPI DPC++ Compiler

DPC++:data parallel C++

*   Set up the environment for oneAPI DPC++ Compiler. For Intel oneAPI Base Toolkit distribution installed to default location you can do this using `setvars.bat` script
    
    **为oneAPI DPC++编译器设置好环境，如果intel oneAPI工具链发行版安装到了默认位置的话，可以用以下命令设置环境**
    
    ```bat
    "C:\Program Files (x86)\Intel\oneAPI\setvars.bat"
    ```
    
    or open `Intel oneAPI Commmand Prompt` instead.
    
    **或者打开这个选项**
    
* Download [oneAPI Level Zero headers](https://github.com/oneapi-src/level-zero/releases/tag/v1.0) from Github and unpack the archive.

*   Generate `Ninja` project
    
    ```bash
    mkdir build
    
    cd build
    
    :: Set C and C++ compilers
    
    set CC=clang
    
    set CXX=clang++
    
    cmake .. -G Ninja -DDNNL_CPU_RUNTIME=DPCPP ^
    
    									-DDNNL_GPU_RUNTIME=DPCPP ^
    
    									-DCMAKE_PREFIX_PATH=<path to Level Zero headers> ^
    
    									<extra build options>
    
    ```
    
    > Note
    >
    > The only CMake generator that supports oneAPI DPC++ Compiler on Windows is Ninja. CC and CXX variables must be set to clang and clang++ respectively.
    >
    > **CMake 生成器唯一支持oneAPI DPC++的编译器是Ninja，CC和CXX必须设置为clang和clang++**
    
* Build the library

  ```bash
  cmake --build
  ```


Validate the Build/检测是否build成功
------------------

If the library is built for the host system, you can run unit tests using:

**如果库是为本机系统安装可以直接运行单元测试**

```bash
ctest
```



Build documenation
------------------

Build the documentation:

**创建相应文档**

```bash
cmake --build . --target doc
```



Install library
---------------

Install the library, headers, and documentation

**安装库文件，头文件还有文档。**

```bash
cmake --build . --target install
```

The install directory is specified by the [CMAKE_INSTALL_PREFIX](https://cmake.org/cmake/help/latest/variable/CMAKE_INSTALL_PREFIX.html) cmake variable. When installing in the default directory, the above command needs to be run with administrative privileges using `sudo` on Linux/Mac or a command prompt run as administrator on Windows.

**安装目录由CMAKE_INSTALL_PREFIX cmake变量指定。 在默认目录中安装时，需要在Linux / Mac上使用sudo以管理员权限运行以上命令，或者在Windows上以管理员身份运行命令提示符。**