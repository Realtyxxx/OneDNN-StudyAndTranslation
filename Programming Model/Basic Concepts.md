# 基本概念
  >  原文地址 [oneapi-src.github.io](https://oneapi-
src.github.io/oneDNN/dev_guide_basic_concepts.html)

### 总结：

> In this page, an outline of the oneDNN programming model is presented, and the key concepts are discu......  
> 这章将会表达oneDNN的编程模型和关键概念

Introduction
------------

In this page, an outline of the oneDNN programming model is presented, and the key concepts are discussed, including _Primitives_, _Engines_, _Streams_, and _Memory Objects_. In essence, the oneDNN programming model consists in executing one or several _primitives_ to process data in one or several _memory objects_. The execution is performed on an _engine_ in the context of a _stream_. The relationship between these entities is briefly presented in Figure 1, which also includes additional concepts relevant to the oneDNN programming model, such as primitive _attributes_ and _descriptors_. These concepts are described below in much more details.  
**这章将会展示oneDNN的编程模型，和关键概念，比如 _原语_，_引擎_，_流_，_内存对象_。oneDNN编程模型包括用一个或多个原语处理一个或者多个内存对象上的数据。这些执行都在_stream_流的上下文中在_engine_引擎中执行，这些实体关系如图一所示，其中还包括了oneDNN编程模型相关的其他概念，比如原始的   _attributes_ （ _属性_ ）和 _descriptor_ （ _描述符_ ）。** 

![](https://oneapi-src.github.io/oneDNN/img_programming_model.png)

Figure 1: Overview of oneDNN programming model. Blue rectangles denote oneDNN objects, and red lines denote dependencies between objects. **图 1oneDNN编程模型的概述，蓝色矩形表示oneDNN对象，红色线条表示对象间依赖。**


### Primitives

* oneDNN is built around the notion of a _primitive_ ([dnnl::primitive](https://oneapi-src.github.io/oneDNN/structdnnl_1_1primitive.html)). A _primitive_ is an object that encapsulates a particular computation such as forward convolution, backward LSTM computations, or a data transformation operation. Additionally, using primitive _attributes_ ([dnnl::primitive_attr](https://oneapi-src.github.io/oneDNN/structdnnl_1_1primitive__attr.html)) certain primitives can represent more complex _fused_ computations such as a forward convolution followed by a ReLU. 
**oneDNN是围绕原语（dnnl :: primitive）的概念构建的。 原语是封装特定计算（例如正向卷积，反向LSTM计算或数据转换操作）的对象。 此外，使用原语属性（dnnl :: primitive_attr），某些原语可以表示更复杂的融合计算，例如后接ReLU的正向卷积。**

* The most important difference between a primitive and a pure function is that a primitive can store state.  
**原语和纯函数之间的最重要区别是，原语可以存储状态。**
* One part of the primitive’s state is immutable. For example, convolution primitives store parameters like tensor shapes and can pre-compute other dependent parameters like cache blocking. This approach allows oneDNN primitives to pre-generate code specifically tailored for the operation to be performed. The oneDNN programming model assumes that the time it takes to perform the pre-computations is amortized by reusing the same primitive to perform computations multiple times.  
**原语的一部分是不可变的。 例如，卷积原语存储类似张量形状的参数，并可以预计算其他相关参数，例如缓存阻塞（缓存打包？cache blocking）。 这种方法允许一个DNN原语预生成专门为将要执行的操作量身定制的代码。 oneDNN编程模型假定通过重复使用同一原语多次执行计算来摊销执行预计算所需的时间。**

* The mutable part of the primitive’s state is referred to as a scratchpad. It is a memory buffer that a primitive may use for temporary storage only during computations. The scratchpad can either be owned by a primitive object (which makes that object non-thread safe) or be an execution-time parameter.
**基本状态的可变部分称为暂存器。 它是一个内存缓冲区，原语仅可在计算期间用于临时存储。 暂存器可以由原始对象拥有（这使该对象成为非线程安全的），也可以是一个执行时参数。**

### Engines

_Engines_ ([dnnl::engine](https://oneapi-src.github.io/oneDNN/structdnnl_1_1engine.html)) is an abstraction of a computational device: a CPU, a specific GPU card in the system, etc. Most primitives are created to execute computations on one specific engine. The only exceptions are reorder primitives that transfer data between two different engines.  
**引擎：其实是计算设备的抽象，有可能是一个CPU或者GPU卡，大部分原语都是在特定引擎上计算，唯一例外是两个不同引擎间传输数据的重排序原语可能会在两个引擎上执行。**

### Streams

_Streams_ ([dnnl::stream](https://oneapi-src.github.io/oneDNN/structdnnl_1_1stream.html)) encapsulate execution context tied to a particular engine. For example, they can correspond to OpenCL command queues.  
**封装执行上下文到一个特定到引擎，一传执行。**

### Memory Objects

_Memory objects_ ([dnnl::memory](https://oneapi-src.github.io/oneDNN/structdnnl_1_1memory.html)) encapsulate handles to memory allocated on a specific engine, tensor dimensions, data type, and memory format – the way tensor indices map to offsets in linear memory space. Memory objects are passed to primitives during execution.  
**内存对象（dnnl :: memory）封装了在特定引擎上分配的内存，张量尺寸,数据类型和内存格式(张量索引映射到线性内存空间中的偏移量的方式)的句柄。 内存对象在执行期间被传递给基元。**

Levels of Abstraction/抽象等级
---------------------  

oneDNN has multiple levels of abstractions for primitives and memory objects in order to expose maximum flexibility to its users.

**oneDNN为了对用户提供最大的灵活性，对于原语和内存对象有多种级别的抽象。**

On the _logical_ level, the library provides the following abstractions:

**在逻辑层次上提供以下抽象：**

* _Memory descriptors_ ([dnnl::memory::desc](https://oneapi-src.github.io/oneDNN/structdnnl_1_1memory_1_1desc.html)) define a tensor's logical dimensions, data type, and the format in which the data is laid out in memory. The special format _any_ ([dnnl::memory::format_tag::any](https://oneapi-src.github.io/oneDNN/structdnnl_1_1memory.html#a8e71077ed6a5f7fb7b3e6e1a5a2ecf3fa100b8cad7cf2a56f6df78f171f97a1ec)) indicates that the actual format will be defined later (see [Memory Format Propagation](https://oneapi-src.github.io/oneDNN/memory_format_propagation_cpp.html)).

* ***内存描述符* ([dnnl::memory::desc](https://oneapi-src.github.io/oneDNN/structdnnl_1_1memory_1_1desc.html)) 定义了张量的逻辑维度、数据类型和数据放置的格式 在内存中。 特殊格式 _any_ ([dnnl::memory::format_tag::any](https://oneapi-src.github.io/oneDNN/structdnnl_1_1memory.html#a8e71077ed6a5f7fb7b3e6e1a5a2ecf3fa100b8cad7cf2a56f6df78f171f97a1ec))表示实际格式将会稍后定义（看[Memory Format Propagation](https://oneapi-src.github.io/oneDNN/memory_format_propagation_cpp.html)）。**

* _Operation descriptors_ (one for each supported primitive) describe an operation's most basic properties without specifying, for example, which engine will be used to compute them. For example, convolution descriptor describes shapes of source, destination, and weights tensors, propagation kind (forward, backward with respect to data or weights), and other implementation-independent parameters.

  **_操作描述符_（每个支持的原语一个）描述了一个操作的最基本属性，但没有指定，例如，将使用哪个引擎来计算它们。 例如，卷积描述符描述了源、目标和权重张量的形状、传播类型（关于数据或权重的前向、后向）以及其他与实现无关的参数。**

* _Primitive descriptors_ ([dnnl_primitive_desc_t](https://oneapi-src.github.io/oneDNN/group__dnnl__api__primitives__common.html#gaabde3e27edf071b62b39f47bace7efd6); in the C++ API there are multiple types for each supported primitive) are at an abstraction level in between operation descriptors and primitives and can be used to inspect details of a specific primitive implementation like expected memory formats via queries to implement memory format propagation (see [Memory Format Propagation](https://oneapi-src.github.io/oneDNN/memory_format_propagation_cpp.html)) without having to fully instantiate a primitive.

  **_原语描述符_（[dnnl_primitive_desc_t](https://oneapi-src.github.io/oneDNN/group__dnnl__api__primitives__common.html#gaabde3e27edf071b62b39f47bace7efd6)；在C++ API中，对每个支持的原语有多种类型）处于操作描述符和原语之间的抽象级别，可用于通过查询来检查特定原语实现的细节，如预期的内存格式，以实现内存格式传播（参见 [内存格式传播](https://oneapi-src.github.io/oneDNN/memory_format_propagation_cpp.html) ) 而不必完全实例化一个原语。**

<table><tbody><tr><th>Abstraction level</th><th>Memory object</th><th>Primitive objects</th></tr><tr><td>Logical description</td><td>Memory descriptor</td><td>Operation descriptor</td></tr><tr><td>Intermediate description</td><td>N/A</td><td>Primitive descriptor</td></tr><tr><td>Implementation</td><td>Memory object</td><td>Primitive</td></tr></tbody></table>

Creating Memory Objects and Primitives/创建内存对象和原语
--------------------------------------

### Memory Objects/内存对象

Memory objects are created from the memory descriptors. It is not possible to create a memory object from a memory descriptor that has memory format set to [dnnl::memory::format_tag::any](https://oneapi-src.github.io/oneDNN/structdnnl_1_1memory.html#a8e71077ed6a5f7fb7b3e6e1a5a2ecf3fa100b8cad7cf2a56f6df78f171f97a1ec "Placeholder memory format tag. ").

**内存对象是由内存描述符创建的，但内存格式设置为[dnnl::memory::format_tag::any](https://oneapi-src.github.io/oneDNN/structdnnl_1_1memory.html#a8e71077ed6a5f7fb7b3e6e1a5a2ecf3fa100b8cad7cf2a56f6df78f171f97a1ec "Placeholder memory format tag. ")的内存描述符是不能创建内存对象的。**

There are two common ways for initializing memory descriptors:

**有两种常用的初始化内存描述符的方法：**

* By using [dnnl::memory::desc](https://oneapi-src.github.io/oneDNN/structdnnl_1_1memory_1_1desc.html) constructors or by extracting a descriptor for a part of a tensor via [dnnl::memory::desc::submemory_desc](https://oneapi-src.github.io/oneDNN/structdnnl_1_1memory_1_1desc.html#a18f16cc91b7b4137aa70f42cf57e32de)

  **通过使用[dnnl::memory::desc](https://oneapi-src.github.io/oneDNN/structdnnl_1_1memory_1_1desc.html)构造函数或者通过[dnnl::memory::desc::submemory_desc](https://oneapi-src.github.io/oneDNN/structdnnl_1_1memory_1_1desc.html#a18f16cc91b7b4137aa70f42cf57e32de)为张量的一部分提取描述符。**

  

* By _querying_ an existing primitive descriptor for a memory descriptor corresponding to one of the primitive's parameters (for example, [dnnl::convolution_forward::primitive_desc::src_desc](https://oneapi-src.github.io/oneDNN/structdnnl_1_1convolution__forward_1_1primitive__desc.html#a0b525cb29996d52ce821b42824589fe3)).

  **通过_查询_与原语参数之一对应的内存描述符的现有原语描述符（例如，[dnnl::convolution_forward::primitive_desc::src_desc](https://oneapi-src.github.io/oneDNN/structdnnl_1_1convolution__forward_1_1primitive__desc.html#a0b525cb29996d52ce821b42824589fe3)).*。

Memory objects can be created with a user-provided handle (a `void *` on CPU), or without one, in which case the library will allocate storage space on its own.

**内存对象可以使用用户提供的句柄（CPU 上的“void *”）创建，也可以不使用，在这种情况下，库将自行分配存储空间。**

### Primitives/原语

The sequence of actions to create a primitive is:

**创建原语的动作序列是：**

1. Create an operation descriptor via, for example, [dnnl::convolution_forward::desc](https://oneapi-src.github.io/oneDNN/structdnnl_1_1convolution__forward_1_1desc.html). The operation descriptor can contain memory descriptors with placeholder [format_tag::any](https://oneapi-src.github.io/oneDNN/structdnnl_1_1memory.html#a8e71077ed6a5f7fb7b3e6e1a5a2ecf3fa100b8cad7cf2a56f6df78f171f97a1ec) memory formats if the primitive supports it.

   **例如，通过 [dnnl::convolution_forward::desc](https://oneapi-src.github.io/oneDNN/structdnnl_1_1convolution__forward_1_1desc.html) 创建操作描述符。 如果支持原语内存格式操作描述符可以包含带有占位符的内存描述符 [format_tag::any](https://oneapi-src.github.io/oneDNN/structdnnl_1_1memory.html#a8e71077ed6a5f7fb7b3e6e1a5a2ecf3fa100b8cad7cf2a56f6df78f171f97a1ec)**。

2. Create a primitive descriptor based on the operation descriptor, engine and attributes.

   **根据操作描述符、引擎和属性创建原始描述符。**

3. Create a primitive based on the primitive descriptor obtained in step 2.

   **根据步骤 2 中获得的基元描述符创建基元。**

Note

The above sequence does not relate to all primitives in its entirety. For instance, the reorder primitive does not have an operation descriptor.

**上述序列并不完全涉及所有原语。 例如，重新排序原语没有操作描述符。**
