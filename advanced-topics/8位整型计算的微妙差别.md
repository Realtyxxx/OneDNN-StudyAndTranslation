# 8位整型计算的微妙差别

> 原文地址 [oneapi-src.github.io](https://oneapi-src.github.io/oneDNN/dev_guide_int8_computations.html)
>
> This document uses **int8** to denote 8-bit integer no matter whether it is signed or unsigned. To emphasize the signedness of the data type **u8** \(`uint8_t`\) or **s8** \(`int8_t`\) are used. In particular, if a primitive has two inputs the types would be written using "/". For instance:
>
> * int8 GEMM denotes any integer GEMM with 8-bit integer inputs, while
> * u8/s8 GEMM denotes [dnnl\_gemm\_u8s8s32\(\)](https://oneapi-src.github.io/oneDNN/group__dnnl__api__blas.html#gaef24848fd198d8a178d3ad95a78c1767) only.
>
> 本文档使用**int8**来表示8位整数，无论它是带符号的还是无符号的。为了强调数据类型的符号性使用了 **u8** \(`uint8_t`\) 或者 **s8** \(`int8_t`\) 。特别的如果一个原语有两个输入，则将使用"/"写入类型。例如
>
> * int8 GEMM表示具有8位整数输入的任何整数GEMM，而
> * u8/s8 GEMM 只表示 [dnnl\_gemm\_u8s8s32\(\)](https://oneapi-src.github.io/oneDNN/group__dnnl__api__blas.html#gaef24848fd198d8a178d3ad95a78c1767) .

The operation primitives that work with the int8 data type \([dnnl::memory::data\_type::s8](https://oneapi-src.github.io/oneDNN/structdnnl_1_1memory.html#a8e83474ec3a50e08e37af76c8c075dcea3e8d88fdd85d7153525e0647cdd97686) and [dnnl::memory::data\_type::u8](https://oneapi-src.github.io/oneDNN/structdnnl_1_1memory.html#a8e83474ec3a50e08e37af76c8c075dcea077393852be20e37026d6281827662f2)\) typically use s32 \(`int32_t`\) as an intermediate data type \([dnnl::memory::data\_type::s32](https://oneapi-src.github.io/oneDNN/structdnnl_1_1memory.html#a8e83474ec3a50e08e37af76c8c075dceaa860868d23f3a68323a2e3f6563d7f31)\) to avoid integer overflows.

使用int8数据类型的操作原语\([dnnl::memory::data\_type::s8](https://oneapi-src.github.io/oneDNN/structdnnl_1_1memory.html#a8e83474ec3a50e08e37af76c8c075dcea3e8d88fdd85d7153525e0647cdd97686) 和 [dnnl::memory::data\_type::u8](https://oneapi-src.github.io/oneDNN/structdnnl_1_1memory.html#a8e83474ec3a50e08e37af76c8c075dcea077393852be20e37026d6281827662f2)\) 通常将 s32 \(`int32_t`\) 用作中间数据类型 \([dnnl::memory::data\_type::s32](https://oneapi-src.github.io/oneDNN/structdnnl_1_1memory.html#a8e83474ec3a50e08e37af76c8c075dceaa860868d23f3a68323a2e3f6563d7f31)\) 来避免整数溢出 .

For instance, the int8 average primitive accumulates the int8 input values in a window to an s32 accumulator, then divides the result by the window size, and then stores the result back to the int8 destination:

例如，int8平均[pooling](https://oneapi-src.github.io/oneDNN/dev_guide_pooling.html)原语将窗口中的int8输入值累加到s32累加器，然后将结果除以窗口大小，然后将结果存储回int8目标：

* $$dst_{s8}(...) = (s8) \Biggl( \Biggl( \sum\limits_{kh,kw} (s32)\src_{s8}(...) \Biggr) \div (kh \cdot kw) \Biggr)$$ 

> Note
>
> The max pooling primitive can directly work with int8 data types.
>
> max pooling原语可以直接与int8数据类型一起使用。

Using an s32 accumulator is especially important for matrix-multiply such as operation primitives that have chains of multiplication and accumulation of int8 values. These primitives are:

使用s32累加器对于矩阵乘法（例如具有int8值的乘法和累加链的操作原语）尤其重要。 这些原语是：

* [Convolution](https://oneapi-src.github.io/oneDNN/dev_guide_convolution.html)

  卷积

* Int8 GEMMs: [dnnl\_gemm\_s8s8s32\(\)](https://oneapi-src.github.io/oneDNN/group__dnnl__api__blas.html#ga2b763b7629846913507d88fba875cc26) and [dnnl\_gemm\_u8s8s32\(\)](https://oneapi-src.github.io/oneDNN/group__dnnl__api__blas.html#gaef24848fd198d8a178d3ad95a78c1767)

  8位整型矩阵乘

* [Inner Product](https://oneapi-src.github.io/oneDNN/dev_guide_inner_product.html)

  内部乘积原语

* [RNN](https://oneapi-src.github.io/oneDNN/dev_guide_rnn.html) with LSTM or GRU cell functions

  使用LSTM\(长短期记忆网络，1997\)和GRU（门控循环单元）单元功能的RNN（循环神经网络）

Ideally, the semantics of these operations should be as follows:

理想情况下，这些操作的语义应如下所示：

1. **Convert all inputs to s32 data type**.

   **将所有输入转换为s32数据类型**。

2. Perform the operation using s32 data type.

   使用s32数据类型（有符号32位数据类型）执行操作。

3. \(Optionally\) [Post process](https://oneapi-src.github.io/oneDNN/dev_guide_attributes_post_ops.html) the data. This typically happens with additional data conversion to the f32 data type.

   （可选） [Post process](https://oneapi-src.github.io/oneDNN/dev_guide_attributes_post_ops.html)数据。 这通常发生在将其他数据转换为f32数据类型的情况下

4. \(Optionally\) Down-convert the result to the destination data type.

   （可选）将结果下转换为目标数据类型。

Depending on the hardware, the first step might vary slightly.

取决于硬件，第一步可能会略有不同。

The data type of computations within a primitive is defined based on the type of the input tensors.

基于输入张量的类型定义图元内计算的数据类型。

This document focuses on the first two steps \(since the last two steps are independent from the hardware used\), and describes the behaviors on different systems and the reasons behind them.

本文档重点介绍前两个步骤（因为后两个步骤与所使用的硬件无关），并描述了不同系统上的行为及其背后的原因。

## CPU

### 1. Inputs of mixed type: u8 and s8

### 1. 输入混合类型：u8 和 s8

Instruction Set Architecture \(ISA\) has special instructions that enable multiplying and adding the vectors of u8 and s8 very efficiently. oneDNN enables int8 support using these particular instructions.

指令集体系结构（ISA）具有特殊的指令，可以非常高效地对u8和s8的向量进行乘法和加法运算。 oneDNN使用这些特定指令启用int8支持。

Unfortunately, these instructions do not have the counterparts that work with vectors of the same type \(either s8/s8 or u8/u8\). The details for the s8/s8 case are covered in the [2. Inputs of the same type: s8](https://oneapi-src.github.io/oneDNN/dev_guide_int8_computations.html#dg_i8_comp_s12) section below.

不幸的是，这些指令没有适用于相同类型向量（s8 / s8或u8 / u8）的对应指令。s8 / s8 例子的详细信息在下面的[2. Inputs of the same type: s8](https://oneapi-src.github.io/oneDNN/dev_guide_int8_computations.html#dg_i8_comp_s12)部分。

#### 1.1. Processors with the Intel AVX2 or Intel AVX-512 Support

#### 1.1 具有Intel AVX2或Intel AVX-512支持的处理器

_System examples: Intel Xeon processor E7 v3 Family \(formerly Haswell\), Intel Xeon Scalable processor x1xx series \(formerly Skylake\)._

_系统示例：Intel Xeon处理器E7 v3家族（以前为Haswell），Intel Xeon可扩展处理器x1xx系列（以前为Skylake）。_

oneDNN implements matrix multiplication such as operations with u8 and s8 operands on the Intel AVX2 and Intel AVX512 Instruction Set by using a sequence of `VPMADDUBSW, VPMADDWD, VPADDD` instructions [\[1\]](https://oneapi-src.github.io/oneDNN/dev_guide_int8_computations.html#dg_i8_ref_sdm):

oneDNN通过使用一系列`VPMADDUBSW，VPMADDWD，VPADDD`指令 [\[1\]](https://oneapi-src.github.io/oneDNN/dev_guide_int8_computations.html#dg_i8_ref_sdm)来实现矩阵乘法，例如在Intel AVX2和Intel AVX512指令集上使用u8和s8操作数进行运算。：

1. `VPMADDUBSW` multiplies two pairs of u8/s8 values and accumulates the result into s16 \(`int16_t`\) with potential saturation.

   `VPMADDUBSW`将两对u8 / s8值相乘，并将结果累加到s16（`int16_t`）中，并具有潜在的饱和度。

2. `VPMADDWD` sums the pairs of s16 values obtained above into s32. Computed sum is exact.

   `VPMADDWD`将上面获得的s16值按对相加到s32中。 计算得出的总和是准确的。

3. `VPADDD` accumulates obtained s32 value to the accumulator.

   `VPADDD`将获得的s32值累加到累加器中。

The pseudo-code for the sequence is shown below:

```text
// Want to compute:
// c_s32 += sum{i=0..3}(a_u8[i] * b_s8[i])
int32_t u8s8s32_compute_avx512(
        uint8_t a_u8[4], int8_t b_s8[4], int32_t c_s32) {
    // Compute using VPMADDUBSW, VPMADDWD, VPADDD:
    int16_t ab_s16[4];
    for (int i = 0; i < 4; ++i)
        ab_s16[i] = (int16_t)a_u8[i] * (int16_t)b_s8[i]; // Exact computations
    int16_t VPMADDUBSW_res[2];
    VPMADDUBSW_res[0] = saturate_to_s16(ab_s16[0] + ab_s16[1]);  // CAUTION: POTENTIAL OVERFLOW / SATURATION
    VPMADDUBSW_res[1] = saturate_to_s16(ab_s16[2] + ab_s16[3]);  // CAUTION: POTENTIAL OVERFLOW / SATURATION
    c_s32 +=
        (int32_t)VPMADDUBSW_res[0] +
        (int32_t)VPMADDUBSW_res[1];
    return c_s32;
}
```

> Note the potential saturation happening at the first step \(or the snippet that is marked with `CAUTION` in the pseudo-code\). Consider the following example:
>
> 注意第一步发生潜在饱和（或在伪代码中标记为`CAUTION`的代码段）。 考虑以下示例：

```text
uint8_t a_u8[4] = {255, 255, 0, 0};
int8_t  b_s8[4] = {127, 127, 0, 0};
int32_t c_s32 = 0;
c_s32 = u8s8s32_compute(a_u8, b_s8, c_s32);
// c_s32 = 32767
//       = 0
//       + max(INT16_MIN, min(INT16_MAX, 255 * 127 + 255 * 127))
//       + max(INT16_MIN, min(INT16_MAX,   0 *   0 +   0 *   0));
// While one might expect 64770 = 255 * 127 + 255 * 127 + 0 * 0 + 0 * 0;
```

This is the major pitfall of using this sequence of instructions. As far as the precise result is concerned, one of the possible instruction sequences would be `VPMOVSXBW/VPMOVZXBW, VPMADDWD, VPADDD` [\[1\]](https://oneapi-src.github.io/oneDNN/dev_guide_int8_computations.html#dg_i8_ref_sdm), where the first ones casts the s8/u8 values to s16. Unfortunately, using them would lead to 2x lower performance.

这是使用此指令序列的主要陷阱。 就精确结果而言，可能的指令序列之一将是`VPMOVSXBW / VPMOVZXBW，VPMADDWD，VPADDD` [\[1\]](https://oneapi-src.github.io/oneDNN/dev_guide_int8_computations.html#dg_i8_ref_sdm)，其中第一个将s8 / u8值转换为s16。 不幸的是，使用它们会导致性能降低2倍。

When one input is of type u8 and the other one is of type s8, oneDNN assumes that it is the user's responsibility to choose the quantization parameters so that no overflow/saturation occurs. For instance, a user can use u7 `[0, 127]` instead of u8 for the unsigned input, or s7 `[-64, 63]` instead of the s8 one. It is worth mentioning that this is required only when the Intel AVX2 or Intel AVX512 Instruction Set is used.

当一个输入的类型为u8而另一输入的类型为s8时，oneDNN认为选择量化参数是用户的责任，因此不会发生溢出/饱和。 例如，用户可以使用u7`[0，127]`代替u8作为无符号输入，或使用s7`[--64，63]`代替s8。 值得一提的是，仅当使用Intel AVX2或Intel AVX512指令集时才需要这样做。

The **RNN** primitive behaves slightly differently than the convolution and inner product primitives, or u8/s8 GEMM. Even though its hidden state is represented by the u8 data type, the non-symmetric quantization is assumed. Namely, the formula is:

**RNN**原语的行为与卷积和内积原语或u8 / s8 GEMM略有不同。 即使其隐藏状态由u8数据类型表示，也可以假定为非对称量化。 即，公式为：

* $$data_{f32}[:] = \frac{1}{scale}(data_{u8}[:] - shift)$$.

But similarly to the other primitives, the RNN primitive does not handle potential overflows automatically. It is up to the user to specify the appropriate quantization parameters \(see [dnnl::primitive\_attr::set\_rnn\_data\_qparams\(\)](https://oneapi-src.github.io/oneDNN/structdnnl_1_1primitive__attr.html#a39ce5aa8b06ed331d8e2158108cc8324) and [dnnl::primitive\_attr::set\_rnn\_weights\_qparams\(\)](https://oneapi-src.github.io/oneDNN/structdnnl_1_1primitive__attr.html#a61bd70f97baa628fd49b2c8b334b913e)\). The recommended ones are:

但是与其他原语类似，RNN原语不会自动处理潜在的溢出。 由用户指定适当的量化参数（请参阅 [dnnl::primitive\_attr::set\_rnn\_data\_qparams\(\)](https://oneapi-src.github.io/oneDNN/structdnnl_1_1primitive__attr.html#a39ce5aa8b06ed331d8e2158108cc8324) 和 [dnnl::primitive\_attr::set\_rnn\_weights\_qparams\(\)](https://oneapi-src.github.io/oneDNN/structdnnl_1_1primitive__attr.html#a61bd70f97baa628fd49b2c8b334b913e)）。 推荐的是：

* Data \(hidden states\) use `scale = 127`, and `shift = 128`.

  数据（隐藏状态）使用 `scale = 127`, and `shift = 128`。

* Weights use `scale = 63 / W_max`, where `W_max` is $max \| W_{f32}\[:\]\| {}_{} $.

  权重使用 `scale = 63 / W_max`, 而 `W_max` 是 $max \| W_{f32}\[:\]\| {}_{} $.

  **1.2. Processors with the Intel DL Boost Support**

  **1.2. 具有Intel DL Boost支持的处理器**

_System examples: Intel Xeon Scalable processor x2xx series \(formerly Cascade Lake\)._

_系统示例：Intel Xeon可扩展处理器x2xx系列（以前为Cascade Lake）。_

Intel DL Boost brings the `VPDPBUSD` instruction [\[2\]](https://oneapi-src.github.io/oneDNN/dev_guide_int8_computations.html#dg_i8_ref_isa_ext), which enables computing the sum of four products of s8 and u8 values. This instruction performs same computations as the sequence of `VPMADDUBSW, VPMADDWD, VPADDD` instructions shown above, but with the major difference that the intermediate overflow and saturation cannot occur.

英特尔DL Boost带来了`VPDPBUSD`指令 [\[2\]](https://oneapi-src.github.io/oneDNN/dev_guide_int8_computations.html#dg_i8_ref_isa_ext)，该指令可以计算s8和u8值的四个乘积之和。 该指令执行与上述`VPMADDUBSW，VPMADDWD，VPADDD`指令序列相同的计算，但主要区别在于不会发生中间溢出和饱和。

In other words, `VPDPBUSD` enables you to exactly compute:

换句话说，`VPDPBUSD`使您能够精确地计算：

```text
// Want to compute:
// c_s32 += sum{i=0..3}(a_u8[i] * b_s8[i])
int32_t u8s8s32_compute_avx512_dl_boost(
        uint8_t a_u8[4], int8_t b_s8[4], int32_t c_s32) {
    // Compute using VPDPBUSD:
    c_s32 +=
        (int32_t)a_u8[0] * (int32_t)b_s8[0] +
        (int32_t)a_u8[1] * (int32_t)b_s8[1] +
        (int32_t)a_u8[2] * (int32_t)b_s8[2] +
        (int32_t)a_u8[3] * (int32_t)b_s8[3];
    return c_s32;
}
```

Since the instruction always computes the result accurately, no special tricks are required, and operations follow the semantics shown above.

由于指令始终准确地计算结果，因此不需要特殊的技巧，并且操作遵循上面显示的语义。

### 2. Inputs of the same type: s8

### 2. 输入相同类型：s8

As mentioned above, with the current instruction set it is impossible to multiply and add two vectors of the s8 data type as efficiently as it is for the mixed case. However, in real-world applications the inputs are typically signed.

如上所述，使用当前指令集，不可能像混合情况一样有效地将s8数据类型的两个向量相乘和相加。 但是，在实际应用中，输入通常是由符号的。

To overcome this issue, oneDNN employs a trick: at run-time, it adds 128 to one of the s8 input to make it of type u8 instead. Once the result is computed, oneDNN subtracts the extra value it added by replacing the s8 with u8. This subtracted value sometimes referred as a **compensation**.

为了克服这个问题，oneDNN使用了一个技巧：在运行时，它向s8输入之一添加128，以使其成为u8类型。 一旦计算出结果，oneDNN便用u8代替s8减去它所增加的额外值。 该减去的值有时称为**补偿**。

Conceptually the formula is:

$$Y_{s32} = X_{s8} \cdot W_{s8} = X'_{u8} \cdot W_{s8} - 128 \cdot W_{s8},$$

where:

* $X'_{u8} = X_{s8} + 128$ , and
* $128 \cdot W_{s8} {}_{} $ is a compensation.

> Note
>
> Since s8/s8 implementations are based on u8/s8 ones, the performance of the former might be slightly lower than the latter. The difference might vary depending on the problem sizes, hardware, and environment, but is expected to be in a range from 0% to 15% in most cases.
>
> 由于s8 / s8实现是基于u8 / s8的实现，因此前者的性能可能会略低于后者。 差异可能会因问题大小，硬件和环境而异，但在大多数情况下，期望范围为0％至15％。
>
> Since s8/s8 implementations are based on u8/s8 ones, they have the same potential issue with overflow/saturation when the Intel AVX2 or Intel AVX512 Instruction Set is used. The difference between the expected and actual results might be much greater though in this case. Consider the following example:
>
> 由于s8 / s8实施基于u8 / s8实施，因此当使用Intel AVX2或Intel AVX512指令集时，它们具有相同的潜在溢出/饱和问题。 在这种情况下，预期结果与实际结果之间的差异可能更大。 考虑以下示例：

```text
int8_t  a_s8[4] = {127, 127, 0, 0};
int8_t  b_s8[4] = {127, 127, 0, 0};
int32_t c_s32 = 0;
// s8s8 the uses u8s8
auto s8s8s32_compute = [](int8_t a_s8[4], int8_t b_s8[4], int32_t c_s32) {
    uint8_t a_u8[4] = { 128 + a_s8[0], ...};
    c_s32 = u8s8s32_compute(a_u8, b_s8, c_s32);
    // apply the compensation
    c_s32 +=
        - 128 * b_s8[0]
        - 128 * b_s8[1]
        - 128 * b_s8[2]
        - 128 * b_s8[3];
    return c_s32;
};
c_s32 = s8s8s32_compute(a_s8, b_s8, c_s32);
// c_s32 = 255
//       = 32767 - 128 * (127 + 127 + 0 + 0);
// While one might expect 32258 !!!
```

Note that processors with no support of the Intel AVX2 and Intel AVX512 Instruction Set or with support of the Intel DL Boost Instruction Set are not affected by these issues due to the reasons described in [1. Inputs of mixed type: u8 and s8](https://oneapi-src.github.io/oneDNN/dev_guide_int8_computations.html#dg_i8_comp_s11) section above.

请注意，由于前面的[1. Inputs of mixed type: u8 and s8](https://oneapi-src.github.io/oneDNN/dev_guide_int8_computations.html#dg_i8_comp_s11) 部分中所述的原因，不支持Intel AVX2和Intel AVX512指令集的处理器或不支持Intel DL Boost指令集的处理器不受这些问题的影响。

Different primitives solve the potential overflow differently. The overview of the implementations are given below:

不同的原语以不同的方式解决潜在的溢出。 下面是实现的概述：

1. **Convolution** primitive. The source is treated as `X_s8`, which would be shifted during the execution. The compensation is precomputed by a reorder during quantization of the weights, and embedded into them. Finally, when the Intel AVX2 or Intel AVX512 Instruction Set is used the reorder additionally scales the weights by 0.5 to overcome the potential overflow issue. During the convolution execution, the result would be re-scaled back. This rescaling introduces an error that might insignificantly affect the inference accuracy \(compared to a platform with the Intel DL Boost Instruction Set\).

   **卷积**原语。 源被视为`X_s8`，在执行过程中会被移位。 补偿在权重量化过程中通过重排预先计算，并嵌入其中。 最后，当使用Intel AVX2或Intel AVX512指令集时，重新排序可以将权重额外缩放0.5，以克服潜在的溢出问题。 在卷积执行期间，结果将重新缩放。 重新缩放后会引入一个错误，该错误可能对推理准确性的影响不大（与具有Intel DL Boost指令集的平台相比）。

2. **s8/s8 GEMM** \([dnnl\_gemm\_s8s8s32\(\)](https://oneapi-src.github.io/oneDNN/group__dnnl__api__blas.html#ga2b763b7629846913507d88fba875cc26)\) does nothing to handle the overflow issue. It is up to the user to prepare the data so that the overflow/saturation does not occur. For instance, the user can specify s7 `[-64, 63]` instead of s8 for the second input.

   **s8/s8 GEMM** \([dnnl\_gemm\_s8s8s32\(\)](https://oneapi-src.github.io/oneDNN/group__dnnl__api__blas.html#ga2b763b7629846913507d88fba875cc26)\) 不能处理溢出问题。 用户应自行准备数据，以免发生溢出/饱和。 例如，用户可以为第二个输入指定s7`[--64，63]`而不是s8。

   > Warning
   >
   > It would not be enough to use s7 `[-64, 63]` for the first input. The only possible way to avoid overflow by shrinking the range of the first input would be to use the range `[-128, -1]`, which is most likely meaningless.
   >
   > 警告
   >
   > 对于第一个输入使用s7`[-64，63]`是不够的。 通过缩小第一个输入的范围来避免溢出的唯一可能方法是使用范围\[\[-128，-1\]\`，这很可能毫无意义。

3. The **inner product** primitive directly calls s8/s8 GEMM, so it inherits the behavior of the latter. The user should consider using the appropriate scaling factors to avoid potential issues.

   **内积**原语直接调用s8 / s8 GEMM，因此它继承了后者的行为。 用户应考虑使用适当的比例因子以避免潜在的问题。

4. The **RNN** primitive does not support s8/s8 inputs.

   **RNN**原语不支持s8 / s8输入。

## GPU

See [Data Types](https://oneapi-src.github.io/oneDNN/dev_guide_data_types.html) for details of int8 data type support on GPU.

## References

\[1\] [Intel\(R\) 64 and IA-32 Architectures Software Developer's Manual Combined Volumes 2A, 2B, 2C, and 2D: Instruction Set Reference, A-Z](https://software.intel.com/content/www/us/en/develop/articles/intel-sdm.html). 325383-070US May 2019.

\[2\] [Intel\(R\) Architecture Instruction Set Extensions and Future Features Programming Reference](https://software.intel.com/content/www/us/en/develop/download/intel-architecture-instruction-set-extensions-and-future-features-programming-reference.html). 319433-037 May 2019. _Chapter 2.1. VPDPBUSD — Multiply and Add Unsigned and Signed Bytes_.

\[3\] Rodriguez, Andres, et al. ["Lower numerical precision deep learning inference and training."](https://software.intel.com/content/www/us/en/develop/articles/lower-numerical-precision-deep-learning-inference-and-training) Intel White Paper \(2018\).

