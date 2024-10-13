# Code generation attributes
# 代码生成属性

>[codegen.md](https://github.com/rust-lang/reference/blob/master/src/attributes/codegen.md)\
>commit: 52e0ff3c11260fb86f19e564684c86560eab4ff9 \
>本章译文最后维护日期：2024-10-13

下述[属性][attributes]用于控制代码生成。

## Optimization hints
## 优化提示

`cold`[属性][attributes]和 `inline`属性给出了某种代码生成方式的提示建议，这种方式可能比没有此提示时更快。这些属性只是提示，可能会被忽略。

这两个属性都可以在[函数][functions]上使用。当这类属性应用于 [trait] 中的函数上时，它们只在那些没有被 trait实现所覆盖的默认函数上生效，而不是所有 trait实现中用到的函数上都生效。这两属性对 trait 中那些没有函数体的函数没有影响。

### The `inline` attribute
### 内联(`inline`)属性

*`inline`[属性][attribute]* 的意义是暗示在调用者(caller)中放置此（属性限定的）函数的副本，而不是在定义此（属性限定的）函数的地方生此函数的代码，然后去让别处代码来调用此函数。

> ***注意***：`rustc` 编译器会根据启发式算法(internal heuristics)[^译者注1]自动内联函数。不正确的内联函数会使程序变慢，所以应该小心使用此属性。

使用内联(`inline`)属性有三种方法：

* `#[inline]` *暗示*执行内联扩展。
* `#[inline(always)]` *暗示*应该一直执行内联扩展。
* `#[inline(never)]` *暗示*应该从不执行内联扩展。

> ***注意***: `#[inline]` 在每种形式中都是一个提示，不是*必须*要在调用者放置此属性限定的函数的副本。

### The `cold` attribute
### `cold`属性

*`cold`[属性][attribute]* 暗示此（属性限定的）函数不太可能被调用。

## The `no_builtins` attribute
## `no_builtins`属性

*`no_builtins`[属性][attribute]* 可以应用在 crate 级别，用以禁用对假定存在的库函数调用的某些代码模式优化。[^译者注2]

## The `target_feature` attribute
## `target_feature`属性

*`target_feature`[属性][attribute]* 可应用于函数上，用来为特定的平台架构特性(platform architecture features)启用该函数的代码生成功能。它使用 [_MetaListNameValueStr_]元项属性句法格式来启用（该平台支持的）特性，但这次要求这个句法里只能有一个 `enable`键，其对应值是一个逗号分隔的由平台特性名字组成的符串。

```rust
# #[cfg(target_feature = "avx2")]
#[target_feature(enable = "avx2")]
unsafe fn foo_avx2() {}
```

每个[目标架构][target architecture]都有一组可以被启用的特性。为不是当前 crate 的编译目标下的CPU架构指定需启用的特性是错误的。

调用启用了当前运行代码的平台不支持的特性编译的函数将导致[未定义行为][undefined behavior]，除非此平台明确声明此调用是安全的。

应用了 `target_feature` 的函数不会内联到不支持给定特性的上下文中。`#[inline(always)]` 属性不能与 `target_feature`属性一起使用。

### Available features
### 可用特性

下面是可用的特性的名称列表。

#### `x86` or `x86_64`

即便是同为 `x86` 或 `x86_64` 平台，并不是所有平台都支持下述特性，而在未启用特定特性的平台下执行带有此平台的特性的代码会导致未定义行为。
因此在这类平台下，`#[target_feature]` 只能应用于[非安全(`unsafe`)函数][unsafe function]。

特性     | 隐式启用 | 描述 | 中文描述
------------|--------------------|-------------------|-------------------
`adx`       |          | [ADX] --- Multi-Precision Add-Carry Instruction Extensions | 多精度进位指令扩展
`aes`       | `sse2`   | [AES] --- Advanced Encryption Standard | 高级加密标准
`avx`       | `sse4.2` | [AVX] --- Advanced Vector Extensions | 高级矢量扩展指令集
`avx2`      | `avx`    | [AVX2] --- Advanced Vector Extensions 2 | 高级矢量扩展指令集2
`bmi1`      |          | [BMI1] --- Bit Manipulation Instruction Sets | 位操作指令集
`bmi2`      |          | [BMI2] --- Bit Manipulation Instruction Sets 2 | 位操作指令集2
`cmpxchg16b`|          | [`cmpxchg16b`] - Compares and exchange 16 bytes (128 bits) of data atomically | 原子地比较和交换16字节（128位）的数据
`f16c`      | `avx`    | [F16C] — 16-bit floating point conversion instructions | 16位浮点转换指令
`fma`       | `avx`    | [FMA3] --- Three-operand fused multiply-add | 三操作乘加指令
`fxsr`      |          | [`fxsave`] and [`fxrstor`] --- Save and restore x87 FPU, MMX Technology, and SSE State | 保存/恢复 x87 FPU、MMX技术，SSE状态
`lzcnt`     |          | [`lzcnt`] --- Leading zeros count  | 前导零计数
`movbe`     |          | [`movbe`] - Move data after swapping bytes | 交换字节后移动数据
`pclmulqdq` | `sse2`   | [`pclmulqdq`] — Packed carry-less multiplication quadword | 压缩的四字（16字节）无进位乘法，主用于加解密处理
`popcnt`    |          | [`popcnt`] --- Count of bits set to 1 | 位1计数，即统计有多少个“为1的位”
`rdrand`    |          | [`rdrand`] --- Read random number | 从芯片上的硬件随机数生成器中获取随机数
`rdseed`    |          | [`rdseed`] --- Read random seed | 从芯片上的硬件随机数生成器中获取为伪随机数生成器设定的种子
`sha`       | `sse2`   | [SHA] --- Secure Hash Algorithm | 安全哈希算法
`sse`       |          | [SSE] --- Streaming <abbr title="Single Instruction Multiple Data">SIMD</abbr> Extensions | 单指令多数据流扩展指令集
`sse2`      | `sse`    | [SSE2] --- Streaming SIMD Extensions 2 | 单指令多数据流扩展指令集2
`sse3`      | `sse2`   | [SSE3] --- Streaming SIMD Extensions 3 | 单指令多数据流扩展指令集3
`sse4.1`    | `ssse3`  | [SSE4.1] --- Streaming SIMD Extensions 4.1 | 单指令多数据流扩展指令集4.1
`sse4.2`    | `sse4.1` | [SSE4.2] --- Streaming SIMD Extensions 4.2 | 单指令多数据流扩展指令集4.2
`ssse3`     | `sse3`   | [SSSE3] --- Supplemental Streaming SIMD Extensions 3 | 增补单指令多数据流扩展指令集3
`xsave`     |          | [`xsave`] --- Save processor extended states | 保存处理器扩展状态
`xsavec`    |          | [`xsavec`] --- Save processor extended states with compaction | 压缩保存处理器扩展状态
`xsaveopt`  |          | [`xsaveopt`] --- Save processor extended states optimized | xsave 指令集的优化版
`xsaves`    |          | [`xsaves`] --- Save processor extended states supervisor | 保存处理器扩展状态监视程序

<!-- 保持各个链接靠近其表格，便于以后的增删改 -->

[ADX]: https://en.wikipedia.org/wiki/Intel_ADX
[AES]: https://en.wikipedia.org/wiki/AES_instruction_set
[AVX]: https://en.wikipedia.org/wiki/Advanced_Vector_Extensions
[AVX2]: https://en.wikipedia.org/wiki/Advanced_Vector_Extensions#AVX2
[BMI1]: https://en.wikipedia.org/wiki/Bit_Manipulation_Instruction_Sets
[BMI2]: https://en.wikipedia.org/wiki/Bit_Manipulation_Instruction_Sets#BMI2
[`cmpxchg16b`]: https://www.felixcloutier.com/x86/cmpxchg8b:cmpxchg16b
[F16C]: https://en.wikipedia.org/wiki/F16C
[FMA3]: https://en.wikipedia.org/wiki/FMA_instruction_set
[`fxsave`]: https://www.felixcloutier.com/x86/fxsave
[`fxrstor`]: https://www.felixcloutier.com/x86/fxrstor
[`lzcnt`]: https://www.felixcloutier.com/x86/lzcnt
[`movbe`]: https://www.felixcloutier.com/x86/movbe
[`pclmulqdq`]: https://www.felixcloutier.com/x86/pclmulqdq
[`popcnt`]: https://www.felixcloutier.com/x86/popcnt
[`rdrand`]: https://en.wikipedia.org/wiki/RdRand
[`rdseed`]: https://en.wikipedia.org/wiki/RdRand
[SHA]: https://en.wikipedia.org/wiki/Intel_SHA_extensions
[SSE]: https://en.wikipedia.org/wiki/Streaming_SIMD_Extensions
[SSE2]: https://en.wikipedia.org/wiki/SSE2
[SSE3]: https://en.wikipedia.org/wiki/SSE3
[SSE4.1]: https://en.wikipedia.org/wiki/SSE4#SSE4.1
[SSE4.2]: https://en.wikipedia.org/wiki/SSE4#SSE4.2
[SSSE3]: https://en.wikipedia.org/wiki/SSSE3
[`xsave`]: https://www.felixcloutier.com/x86/xsave
[`xsavec`]: https://www.felixcloutier.com/x86/xsavec
[`xsaveopt`]: https://www.felixcloutier.com/x86/xsaveopt
[`xsaves`]: https://www.felixcloutier.com/x86/xsaves

#### `aarch64`

该目标平台要求 `#[target_feature]`属性仅适用于 [`unsafe`函数][unsafe function]

关于这些特性的更多文档可以在 [developer.arm.com] 上的 [ARM架构参考手册][ARM Architecture Reference Manual]中或 [developer.arm.com] 上的其他地方找到。
[ARM Architecture Reference Manual]: https://developer.arm.com/documentation/ddi0487/latest
[developer.arm.com]: https://developer.arm.com

> ***注意***: 如果要使用的话，`paca` 和 `pacg` 这对特性应同时标记为启用或禁用，因为目前 LLVM 将其作为一个特性来实现的。

特性        | 隐式启用 | 特性名称
---------------|--------------------|-------------------
`aes`          | `neon`         | FEAT_AES & FEAT_PMULL - 高级 <abbr title="Single Instruction Multiple Data">SIMD</abbr> AES & PMULL 指令
`bf16`         |                | FEAT_BF16 - BFloat16 指令
`bti`          |                | FEAT_BTI - 分支目标识别
`crc`          |                | FEAT_CRC - CRC32 *校验和*指令
`dit`          |                | FEAT_DIT - 与数据无关的定时指令
`dotprod`      |                | FEAT_DotProd - Advanced SIMD Int8 dot product instructions
`dpb`          |                | FEAT_DPB - 数据持久点之后的缓存清理
`dpb2`         |                | FEAT_DPB2 - 数据深度持久点之后的缓存清理
`f32mm`        | `sve`          | FEAT_F32MM - SVE单精度 FP矩阵乘法指令
`f64mm`        | `sve`          | FEAT_F64MM - SSVE双精度 FP矩阵乘法指令
`fcma`         | `neon`         | FEAT_FCMA - 浮点复数支持
`fhm`          | `fp16`         | FEAT_FHM - 半精度 FP FMLAL 指令
`flagm`        |                | FEAT_FlagM - 条件标志操作
`fp16`         | `neon`         | FEAT_FP16 - 半精度 FP数据处理
`frintts`      |                | FEAT_FRINTTS - 浮点到整型的辅助转换指令
`i8mm`         |                | FEAT_I8MM - Int8 的矩阵乘法
`jsconv`       | `neon`         | FEAT_JSCVT - JavaScript 转换指令
`lse`          |                | FEAT_LSE - Large System Extension
`lor`          |                | FEAT_LOR - Limited Ordering Regions extension
`mte`          |                | FEAT_MTE & FEAT_MTE2 - 内存标记扩展
`neon`         |                | FEAT_FP & FEAT_AdvSIMD - 浮点和高级SIMD扩展
`pan`          |                | FEAT_PAN - Privileged Access-Never extension
`paca`         |                | FEAT_PAuth - 指针身份验证（地址身份验证）
`pacg`         |                | FEAT_PAuth - 指针身份验证（通用身份验证）
`pmuv3`        |                | FEAT_PMUv3 - 性能监视器扩展（v3）
`rand`         |                | FEAT_RNG - 随机数发生器
`ras`          |                | FEAT_RAS & FEAT_RASv1p1 - 可靠性、可用性和可维护性扩展
`rcpc`         |                | FEAT_LRCPC - Release consistent Processor Consistent
`rcpc2`        | `rcpc`         | FEAT_LRCPC2 - 带即时偏移的rcpc
`rdm`          |                | FEAT_RDM - Rounding Double Multiply accumulate
`sb`           |                | FEAT_SB - Speculation Barrier
`sha2`         | `neon`         | FEAT_SHA1 & FEAT_SHA256 - 高级 SIMD SHA 指令
`sha3`         | `sha2`         | FEAT_SHA512 & FEAT_SHA3 - 高级 SIMD SHA 指令
`sm4`          | `neon`         | FEAT_SM3 & FEAT_SM4 - 高级 SIMD SM3/4 指令
`spe`          |                | FEAT_SPE - 统计分析扩展
`ssbs`         |                | FEAT_SSBS & FEAT_SSBS2 - Speculative Store Bypass Safe
`sve`          | `fp16`         | FEAT_SVE - 可伸缩向量扩展
`sve2`         | `sve`          | FEAT_SVE2 - 可伸缩向量扩展2
`sve2-aes`     | `sve2`, `aes`  | FEAT_SVE_AES - SVE AES 指令
`sve2-sm4`     | `sve2`, `sm4`  | FEAT_SVE_SM4 - SVE SM4 指令
`sve2-sha3`    | `sve2`, `sha3` | FEAT_SVE_SHA3 - SVE SHA3 指令
`sve2-bitperm` | `sve2`         | FEAT_SVE_BitPerm - SVE位置换
`tme`          |                | FEAT_TME - 事务内存扩展
`vh`           |                | FEAT_VHE - 虚拟化主机扩展

#### `riscv32` or `riscv64`

此类目标平台要求 `#[target_feature]`属性只能应用在 [`unsafe` 函数][unsafe function]上。

有关这些功能的进一步文档可以在其各自的规范中找到。可以在 [RISC-V ISA手册][RISC-V ISA Manual]或 [RISC-V GitHub账户][RISC-V GitHub Account]上的手册中参阅相关规范细节。

[RISC-V ISA Manual]: https://github.com/riscv/riscv-isa-manual
[RISC-V GitHub Account]: https://github.com/riscv

特性     | 隐式启用  | 描述
------------|---------------------|-------------------
`a`         |                     | [A][rv-a] --- 原子指令
`c`         |                     | [C][rv-c] --- 压缩指令
`m`         |                     | [M][rv-m] --- 整数乘除法指令
`zb`        | `zba`, `zbc`, `zbs` | [Zb][rv-zb] --- 位操作指令
`zba`       |                     | [Zba][rv-zb-zba] --- 地址生成指令
`zbb`       |                     | [Zbb][rv-zb-zbb] — 基本位操作
`zbc`       |                     | [Zbc][rv-zb-zbc] --- 无进位乘法指令
`zbkb`      |                     | [Zbkb][rv-zb-zbkb] --- 加密算法下的位操作指令
`zbkc`      |                     | [Zbkc][rv-zb-zbc] --- 加密算法下的无进位乘法指令
`zbkx`      |                     | [Zbkx][rv-zb-zbkx] --- 交叉排列 
`zbs`       |                     | [Zbs][rv-zb-zbs] — 单比特指令
`zk`        | `zkn`, `zkr`, `zks`, `zkt`, `zbkb`, `zbkc`, `zkbx` | [Zk][rv-zk] --- 标量加密
`zkn`       | `zknd`, `zkne`, `zknh`, `zbkb`, `zbkc`, `zkbx`     | [Zkn][rv-zkn] --- NIST算法套件扩展
`zknd`      |                                                    | [Zknd][rv-zknd] --- NIST算法套件: AES解密
`zkne`      |                                                    | [Zkne][rv-zkne] --- NIST算法套件: AES加密
`zknh`      |                                                    | [Zknh][rv-zknh] --- NIST算法套件: 哈希函数指令
`zkr`       |                                                    | [Zkr][rv-zkr] --- 熵源扩展
`zks`       | `zksed`, `zksh`, `zbkb`, `zbkc`, `zkbx`            | [Zks][rv-zks] --- ShangMi算法套件
`zksed`     |                                                    | [Zksed][rv-zksed] --- ShangMi算法套件: SM4分组密码指令
`zksh`      |                                                    | [Zksh][rv-zksh] --- ShangMi算法套件: SM3哈希函数指令
`zkt`       |                                                    | [Zkt][rv-zkt] --- 数据独立执行延迟子集

<!-- Keep links near each table to make it easier to move and update. -->

[rv-a]: https://github.com/riscv/riscv-isa-manual/blob/de46343a245c6ee1f7b1a40c92fe1a86bd4f4978/src/a-st-ext.adoc
[rv-c]: https://github.com/riscv/riscv-isa-manual/blob/de46343a245c6ee1f7b1a40c92fe1a86bd4f4978/src/c-st-ext.adoc
[rv-m]: https://github.com/riscv/riscv-isa-manual/blob/de46343a245c6ee1f7b1a40c92fe1a86bd4f4978/src/m-st-ext.adoc
[rv-zb]: https://github.com/riscv/riscv-bitmanip
[rv-zb-zba]: https://github.com/riscv/riscv-bitmanip/blob/main/bitmanip/zba.adoc
[rv-zb-zbb]: https://github.com/riscv/riscv-bitmanip/blob/main/bitmanip/zbb.adoc
[rv-zb-zbc]: https://github.com/riscv/riscv-bitmanip/blob/main/bitmanip/zbc.adoc
[rv-zb-zbkb]: https://github.com/riscv/riscv-bitmanip/blob/main/bitmanip/zbkb.adoc
[rv-zb-zbkc]: https://github.com/riscv/riscv-bitmanip/blob/main/bitmanip/zbkc.adoc
[rv-zb-zbkx]: https://github.com/riscv/riscv-bitmanip/blob/main/bitmanip/zbkx.adoc
[rv-zb-zbs]: https://github.com/riscv/riscv-bitmanip/blob/main/bitmanip/zbs.adoc
[rv-zk]: https://github.com/riscv/riscv-crypto/blob/e2dd7d98b7f34d477e38cb5fd7a3af4379525189/doc/scalar/riscv-crypto-scalar-zk.adoc
[rv-zkn]: https://github.com/riscv/riscv-crypto/blob/e2dd7d98b7f34d477e38cb5fd7a3af4379525189/doc/scalar/riscv-crypto-scalar-zkn.adoc
[rv-zkne]: https://github.com/riscv/riscv-crypto/blob/e2dd7d98b7f34d477e38cb5fd7a3af4379525189/doc/scalar/riscv-crypto-scalar-zkne.adoc
[rv-zknd]: https://github.com/riscv/riscv-crypto/blob/e2dd7d98b7f34d477e38cb5fd7a3af4379525189/doc/scalar/riscv-crypto-scalar-zknd.adoc
[rv-zknh]: https://github.com/riscv/riscv-crypto/blob/e2dd7d98b7f34d477e38cb5fd7a3af4379525189/doc/scalar/riscv-crypto-scalar-zknh.adoc
[rv-zkr]: https://github.com/riscv/riscv-crypto/blob/e2dd7d98b7f34d477e38cb5fd7a3af4379525189/doc/scalar/riscv-crypto-scalar-zkr.adoc
[rv-zks]: https://github.com/riscv/riscv-crypto/blob/e2dd7d98b7f34d477e38cb5fd7a3af4379525189/doc/scalar/riscv-crypto-scalar-zks.adoc
[rv-zksed]: https://github.com/riscv/riscv-crypto/blob/e2dd7d98b7f34d477e38cb5fd7a3af4379525189/doc/scalar/riscv-crypto-scalar-zksed.adoc
[rv-zksh]: https://github.com/riscv/riscv-crypto/blob/e2dd7d98b7f34d477e38cb5fd7a3af4379525189/doc/scalar/riscv-crypto-scalar-zksh.adoc
[rv-zkt]: https://github.com/riscv/riscv-crypto/blob/e2dd7d98b7f34d477e38cb5fd7a3af4379525189/doc/scalar/riscv-crypto-scalar-zkr.adoc
[rv-zkt]: https://github.com/riscv/riscv-crypto/blob/e2dd7d98b7f34d477e38cb5fd7a3af4379525189/doc/scalar/riscv-crypto-scalar-zkt.adoc

#### `wasm32` or `wasm64`

在这两个平台下，安全函数和[非安全函数][unsafe function]均可启用 `#[target_feature]`特性。不可能经由 `#[target_feature]`特性导致未定义行为，因为尝试使用 Wasm引擎不支持的指令将在加载时就失败，而不会有被以不同于编译器预期的方式来解释编译后的代码的风险。

特性     | 描述
------------|-------------------
`bulk-memory`         | [WebAssembly 大容量内存操作提案][bulk-memory]
`extended-const`      | [WebAssembly 扩展常量表达式提案][extended-const]
`mutable-globals`     | [WebAssembly 可变全局变量提案][mutable-globals]
`nontrapping-fptoint` | [WebAssembly 非捕获式浮点到整型转换提案][nontrapping-fptoint]
`sign-ext`            | [WebAssembly 有符号型扩展运算符提案][sign-ext]
`simd128`             | [WebAssembly simd 提案][simd128]

[bulk-memory]: https://github.com/WebAssembly/bulk-memory-operations
[extended-const]: https://github.com/WebAssembly/extended-const
[mutable-globals]: https://github.com/WebAssembly/mutable-global
[nontrapping-fptoint]: https://github.com/WebAssembly/nontrapping-float-to-int-conversions
[sign-ext]: https://github.com/WebAssembly/sign-extension-ops
[simd128]: https://github.com/webassembly/simd

### Additional information
### 附加信息

请参阅 [`target_feature`-条件编译选项][`target_feature` conditional compilation option]，了解如何基于编译时的设置来有选择地启用或禁用对某些代码的编译。注意，条件编译选项不受 `target_feature`属性的影响，只是被整个 crate 启用的特性所驱动。

请参阅标准库中的 [`is_x86_feature_detected`] 或 [`is_aarch64_feature_detected`] 这两个宏，它们可以用来检测平台上的运行时特性。

> 注意：`rustc` 为每个编译目标和 CPU 启用了一组默认特性。编译时，可以使用命令行参数 [`-C target-cpu`] 选择目标 CPU。可以通过命令行参数 [`-C target-feature`] 来为整个 crate 启用或禁用某些单独的特性。

## The `track_caller` attribute
## `track_caller`属性

`track_caller`属性可以应用于除程序入口函数 `fn main` 之外的任何带有 [`"Rust"` ABI][rust-abi] 的函数。当此属性应用于 trait声明中的函数或方法时，该属性将应用在其所有的实现上。如果 trait 本身提供了带有该属性的默认函数实现，那么该属性也应用于其覆盖实现(override implementations)。

当应用于外部(`extern`)块中的函数上时，该属性也必须应用于此函数的任何链接实现(linked implementations)上，否则将导致未定义行为。当此属性应用在一个外部(`extern`)块内可用的函数上时，该外部(`extern`)块中的对该函数的声明也必须带上此属性，否则将导致未定义行为。

### Behavior
### 表现

将此属性应用到函数 `f` 上将允许 `f` 内的代码获得 `f` 被调用时建立的调用栈的“最顶层”的调用的[位置(`Location`)][`Location`]信息的提示。从观察的角度来看，此属性的实现表现地就像从 `f` 所在的帧向上遍历调用栈，定位找到最近的有*非此属性限定*的调用函数 `outer`，并返回 `outer` 调用时的[位置(`Location`)][`Location`]信息。

```rust
#[track_caller]
fn f() {
    println!("{}", std::panic::Location::caller());
}
```

> 注意：`core` 提供了 [`core::panic::Location::caller`] 来观察调用者的位置。它封装(wrap)了由 `rustc` 实现的内部函数(intrinsic) [`core::intrinsics::caller_location`]。

> 注意：由于结果 `Location` 是一个提示，所以具体实现可能会提前终止对堆栈的遍历。请参阅[限制](#limitations)以了解重要的注意事项。

#### Examples
#### 示例

当 `f` 直接被 `calls_f` 调用时，`f` 中的代码观察其在 `calls_f` 内的调用位置：

```rust
# #[track_caller]
# fn f() {
#     println!("{}", std::panic::Location::caller());
# }
fn calls_f() {
    f(); // <-- f() 将打印此处的位置信息
}
```

`f` 被另一个有此属性限定的函数 `g` 调用，`g` 又被 `calls_g`' 调用，`f` 和 `g` 内的代码又同时观察 `g` 在 `calls_g` 内的调用位置：

```rust
# #[track_caller]
# fn f() {
#     println!("{}", std::panic::Location::caller());
# }
#[track_caller]
fn g() {
    println!("{}", std::panic::Location::caller());
    f();
}

fn calls_g() {
    g(); // <-- g() 将两次打印此处的位置信息，一次是它自己，一次是此 f() 里来的
}
```

当`g` 又被另一个有此属性限定的函数 `h` 调用，而`g` 又被 `calls_h`' 调用，`f`、`g` 和 `h` 内的代码又同时观察 `h` 在 `calls_h` 内的调用位置：

```rust
# #[track_caller]
# fn f() {
#     println!("{}", std::panic::Location::caller());
# }
# #[track_caller]
# fn g() {
#     println!("{}", std::panic::Location::caller());
#     f();
# }
#[track_caller]
fn h() {
    println!("{}", std::panic::Location::caller());
    g();
}

fn calls_h() {
    h(); // <-- 将三次打印此处的位置信息，一次是它自己，一次是此 g() 里来，一次是从 f() 里来的
}
```

以此类推。

### Limitations
### 限制

`track_caller`属性获取的信息是只是一个提示信息，实现不需要维护它。

特别是，将带有 `#[track_caller]` 的函数自动强转为函数指针会创建一个填充对象，该填充对象在观察者看来似乎是在此(属性限定的)函数的定义处调用的，从而在这层虚拟调用中丢失了实际的调用者信息。这种自动强转情况的一个常见示例是创建方法被此属性限定的 trait对象[^译者注3] 。

> 注意：前面提到的函数指针填充对象是必需的，因为 `rustc` 会通过向函数的 ABI 附加一个隐式参数来实现代码生成(codegen)上下文中的 `track_caller`，但这种添加是不健壮的(unsound)，因为该隐式参数不是函数类型的一部分，那给定的函数指针类型可能引用也可能不引用具有此属性的函数。这里创建一个填充对象会对函数指针的调用方隐藏隐式参数，从而保持可靠性。

[^译者注1]: 可字面理解为内部反复试探。

[^译者注2]: 默认情况下，Rust 编译器会默认某些标准库函数在编译时可用，编译器也会把当前编译的代码往这些库函数可用的方向去优化。

[^译者注3]: 因为 trait对象的值不能直接使用，只能自动强转为指针引用，那这里的调用就无法观察到真实的调用位置。

[_MetaListNameValueStr_]: ../attributes.md#meta-item-attribute-syntax
[`-C target-cpu`]: https://doc.rust-lang.org/rustc/codegen-options/index.html#target-cpu
[`-C target-feature`]: https://doc.rust-lang.org/rustc/codegen-options/index.html#target-feature
[`is_x86_feature_detected`]: .https://doc.rust-lang.org/std/arch/macro.is_x86_feature_detected.html
[`is_aarch64_feature_detected`]: https://doc.rust-lang.org/std/arch/macro.is_aarch64_feature_detected.html
[`target_feature` conditional compilation option]: ../conditional-compilation.md#target_feature
[attribute]: ../attributes.md
[attributes]: ../attributes.md
[functions]: ../items/functions.md
[target architecture]: ../conditional-compilation.md#target_arch
[trait]: ../items/traits.md
[undefined behavior]: ../behavior-considered-undefined.md
[unsafe function]: ../unsafe-keyword.md
[rust-abi]: ../items/external-blocks.md#abi
[`Location`]: core::panic::Location

## The `instruction_set` attribute
## `instruction_set`属性

*`instruction_set`[属性][attribute]* 可以应用于函数，用以控制将为哪个指令集生成函数。
这允许在单个程序中混合使用多个它支持的 CPU架构指令集。
它使用 [_MetaListPath_]语法，以及由体系结构系列名称和指令集名称组成的路径。

[_MetaListPath_]: ../attributes.md#meta-item-attribute-syntax

在不支持的 `instruction_set`属性的目标架构上使用该属性会报编译错误。

### On ARM

在 `ARMv4T` 和 `ARMv5te` 架构下， 支持使用:

* `arm::a32` - 生成 A32 "ARM" 指令风格的函数。
* `arm::t32` - 生成 T32 "Thumb" 指令风格的函数

<!-- ignore: arm-only -->
```rust,ignore
#[instruction_set(arm::a32)]
fn foo_arm_code() {}

#[instruction_set(arm::t32)]
fn bar_thumb_code() {}
```

使用 `instruction_set`属性会带来以下副作用：

*如果将函数的地址作为函数指针，则地址的低位将根据指令集设置为 0（arm）或 1（thumb）。
*函数中的任何内联程汇编指令都必须使用指定的指令集，而不是目标默认值。