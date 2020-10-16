# 代码生成属性

>[codegen.md](https://github.com/rust-lang/reference/blob/master/src/attributes/codegen.md)\
>commit bc1a70875865bd578b473a431fb66788fb635886

下述[属性]用于控制代码生成。

## 优化提示

`cold` 和 `inline`[属性]给出了相关生成代码方式的建议，这种方法可能比没有提示时更快。属性只是提示，可能会被忽略。

这两个属性都可以在[函数]上使用。当应用于一个[trait]中的一个函数时，它们只应用于trait实现未覆盖此默认函数的情况，而不是所有trait实现。如果 trait 中，相关函数只有声明，没有实现，那这些属性对trait 函数没任何影响。

### `inline`属性

*`inline`[属性]*建议在调用者中放置此(属性限定的)函数的副本，而不是在定义函数的地方生成调用此函数的代码。

> ***注意***：`rustc` 编译器会根据启发式算法（internal heuristics，译者注：可字面理解为内部试探）自动内联函数。不正确的内联函数会使程序变慢，所以应该小心使用此属性。

使用 inline属性有三种方法：

* `#[inline]` *建议*执行内联扩展。
* `#[inline(always)]` *建议*应该一直执行内联扩展。
* `#[inline(never)]` *建议*应该从不执行内联扩展。

> ***注意***: `#[inline]` 在每种形式中都是一个提示，在 Rust 中不*需要*在调用者中放置此(属性限定的)函数的副本。

### `cold`属性

*`cold`[属性]*表示此(属性限定的)函数不太可能被调用。

## `no_builtins`属性

*`no_builtins`[属性]*可以应用在 crate 级别，用以禁止在某些代码模式下对假定存在的库函数的调用的优化。

## `target_feature`属性

*`target_feature`[属性]*可应用于[非安全函数]，用来为特定的平台架构特性（platform architecture features）生成该函数的代码。它使用[_MetaListNameValueStr_]句法规则来启用（该平台支持的）特性，这个规则中一个属性名是 `enable` ，对应值是一个逗号分隔的由平台特性名字组成的符串。

```rust
# #[cfg(target_feature = "avx2")]
#[target_feature(enable = "avx2")]
unsafe fn foo_avx2() {}
```

每个[目标架构]都有一组可能被启用的特性。为不是当前 crate 的编译目标下的CPU架构指定启用特性是错误的。

调用一个编译时启用了某特性的函数，但当前运行代码的平台并不支持该特性，那这将导致[未定义行为]。

应用了 `target_feature` 的函数不会内联到不支持给定特性的上下文中。`#[inline(always)]` 属性不能与 `target_feature`属性一起使用。

### 可用特性

下面是可用特性列表。

#### `x86` 或 `x86_64`

特性     | 隐式启用 | 描述 | 中文描述
------------|--------------------|-------------------|-------------------
`aes`       | `sse2`   | [AES] — Advanced Encryption Standard | 高级加密标准
`avx`       | `sse4.2` | [AVX] — Advanced Vector Extensions | 高级矢量扩展指令集
`avx2`      | `avx`    | [AVX2] — Advanced Vector Extensions 2 | 高级矢量扩展指令集2
`bmi1`      |          | [BMI1] — Bit Manipulation Instruction Sets | 位操作指令集
`bmi2`      |          | [BMI2] — Bit Manipulation Instruction Sets 2 | 位操作指令集2
`fma`       | `avx`    | [FMA3] — Three-operand fused multiply-add | 三操作乘加指令
`fxsr`      |          | [`fxsave`] and [`fxrstor`] — Save and restore x87 FPU, MMX Technology, and SSE State | 保存/恢复 x87 FPU、MMX技术，SSE状态
`lzcnt`     |          | [`lzcnt`] — Leading zeros count  | 前导零计数
`pclmulqdq` | `sse2`   | [`pclmulqdq`] — Packed carry-less multiplication quadword | 压缩的四字（16字节）无进位乘法，主用于加解密处理
`popcnt`    |          | [`popcnt`] — Count of bits set to 1 | 位1计数，即统计有多少个“为1的位”
`rdrand`    |          | [`rdrand`] — Read random number | 从芯片上的硬件随机数生成器中获取随机数
`rdseed`    |          | [`rdseed`] — Read random seed | 从芯片上的硬件随机数生成器中获取为伪随机数生成器设定的种子
`sha`       | `sse2`   | [SHA] — Secure Hash Algorithm | 安全哈希算法
`sse`       |          | [SSE] — Streaming <abbr title="Single Instruction Multiple Data">SIMD</abbr> Extensions | 单指令多数据流扩展指令集
`sse2`      | `sse`    | [SSE2] — Streaming SIMD Extensions 2 | 单指令多数据流扩展指令集2
`sse3`      | `sse2`   | [SSE3] — Streaming SIMD Extensions 3 | 单指令多数据流扩展指令集3
`sse4.1`    | `ssse3`  | [SSE4.1] — Streaming SIMD Extensions 4.1 | 单指令多数据流扩展指令集4.1
`sse4.2`    | `sse4.1` | [SSE4.2] — Streaming SIMD Extensions 4.2 | 单指令多数据流扩展指令集4.2
`ssse3`     | `sse3`   | [SSSE3] — Supplemental Streaming SIMD Extensions 3 | 增补单指令多数据流扩展指令集3
`xsave`     |          | [`xsave`] — Save processor extended states | 保存处理器扩展状态
`xsavec`    |          | [`xsavec`] — Save processor extended states with compaction | 压缩保存处理器扩展状态
`xsaveopt`  |          | [`xsaveopt`] — Save processor extended states optimized | xsave 指令集的优化版
`xsaves`    |          | [`xsaves`] — Save processor extended states supervisor | 保存处理器扩展状态监视程序

<!-- 保持各个链接靠近其表格，便于以后的增删改 -->

[AES]: https://en.wikipedia.org/wiki/AES_instruction_set
[AVX]: https://en.wikipedia.org/wiki/Advanced_Vector_Extensions
[AVX2]: https://en.wikipedia.org/wiki/Advanced_Vector_Extensions#AVX2
[BMI1]: https://en.wikipedia.org/wiki/Bit_Manipulation_Instruction_Sets
[BMI2]: https://en.wikipedia.org/wiki/Bit_Manipulation_Instruction_Sets#BMI2
[FMA3]: https://en.wikipedia.org/wiki/FMA_instruction_set
[`fxsave`]: https://www.felixcloutier.com/x86/fxsave
[`fxrstor`]: https://www.felixcloutier.com/x86/fxrstor
[`lzcnt`]: https://www.felixcloutier.com/x86/lzcnt
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

### 附加信息

请参阅 [`target_feature`-条件编译选项]，了解如何基于编译时的设置来有选择地启用或禁用编译选项。注意，编译选项不受 `target_feature`属性的影响，只为整个 crate 启用的特性所驱动。

有关x86平台上的运行时特性检测，请参阅标准库中的 [`is_x86_feature_detected`] 宏。

> 注意：`rustc`为每个目标平台和CPU启用了一组默认特性。可以使用 [`-C target-cpu`] 标志选择CPU。单个特性可以通过 [`-C target-feature`] 标志来为整个 crate 启用或禁用。

## `track_caller`属性

`track_caller` 属性可以应用于任何带有 [`"Rust"` ABI][rust-abi] 的函数，但程序入口函数 `fn main` 除外。当应用于 crate声明中的函数或方法时，该属性将应用于其所有实现。如果 crate 提供了带有该属性的默认实现，那么该属性也应用于其覆盖实现。

<!-- 当应用于 `extern`块中的函数时，该属性还必须应用于任何此函数的实现，否则将导致未定义行为。当应用于可用于 `extern`块中的函数时，`extern`块中的声明也必须带上此属性，否则将导致未定义行为。When applied to a function which is made available to an `extern` block, the declaration in the `extern` block must also have the attribute, otherwise undefined behavior results. TobeModify-->

### 表现

将此属性应用到函数 `f` 上将允许 `f` 中的代码获得 `f` 被调用时建立的调用栈里“最顶层”的调用 [`Location`] 信息的提示。从观察的角度来看，此属性的实现表现地就像从 `f` 的帧向上遍历堆栈，定位找到最近的有*非此属性限定*的函数 `outer`，并返回 `outer` 的调用 [`Location`] 信息。

```rust
#[track_caller]
fn f() {
    println!("{}", std::panic::Location::caller());
}
```

> 注意：`core` 提供 [`core::panic::Location::caller`] 来观察调用者的位置。它封装了由 `rustc` 实现的内部函数 [`core::intrinsics::caller_location`]。

> 注意：由于结果 `Location` 是一个提示，所以具体实现可能会提前终止对堆栈的遍历。请参阅 [限制](#限制) 以获得重要的注意事项。

#### 示例

当 `f` 直接被 `calls_f` 调用时，`f` 中的代码观察其在`calls_f` 内的调用位置：

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

### 限制

此信息是一个提示，不需要实现来保存它。
<!-- This information is a hint and implementations are not required to preserve it. TobeModify-->

特别是，将带有 `#[track_caller]` 的函数自动强转为函数指针会创建一个填充程序，在观察者看来该填充程序似乎是在此(属性限定的)函数的定义处调用的，从而在虚拟调用中丢失实际的调用者信息。这种自动强转情况的一个常见示例是创建一个 trait对象，而该对象的方法被此属性限定。
<!-- In particular, coercing a function with `#[track_caller]` to a function pointer creates a shim which appears to observers to have been called at the attributed function's definition site, losing actual caller information across virtual calls. A common example of this coercion is the creation of a trait object whose methods are attributed. TobeModify-->

> 注意：前面提到的函数指针填充程序是必需的，因为 `rustc` 会通过向函数的ABI附加一个隐式参数来实现 codegen上下文中的 `track_caller`，但对于间接调用来说，这是不健全的，因为参数不是函数类型的一部分，给定的函数指针类型可能引用也可能不引用具有此属性的函数。填充程序的创建会对函数指针的调用方隐藏隐式参数，从而保持可靠性。
<!-- > Note: The aforementioned shim for function pointers is necessary because `rustc` implements `track_caller` in a codegen context by appending an implicit parameter to the function ABI, but this would be unsound for an indirect call because the parameter is not a part of the function's type and a given function pointer type may or may not refer to a function with the attribute. The creation of a shim hides the implicit parameter from callers of the function pointer, preserving soundness. TobeModify-->

[_MetaListNameValueStr_]: ../attributes.md#元项属性句法
[`-C target-cpu`]: https://doc.rust-lang.org/rustc/codegen-options/index.html#target-cpu
[`-C target-feature`]: https://doc.rust-lang.org/rustc/codegen-options/index.html#target-feature
[`is_x86_feature_detected`]: https://doc.rust-lang.org/std/macro.is_x86_feature_detected.html
[`target_feature`-条件编译选项]: ../conditional-compilation.md#target_feature
[属性]: ../attributes.md
[函数]: ../items/functions.md
[目标架构]: ../conditional-compilation.md#target_arch
[trait]: ../items/traits.md
[未定义表现]: ../behavior-considered-undefined.md
[非安全函数]: ../unsafe-functions.md
[rust-abi]: ../items/external-blocks.md#abi
[`core::intrinsics::caller_location`]: https://doc.rust-lang.org/core/intrinsics/fn.caller_location.html
[`core::panic::Location::caller`]: https://doc.rust-lang.org/core/panic/struct.Location.html#method.caller
[`Location`]: https://doc.rust-lang.org/core/panic/struct.Location.html
<!-- 2020-10-16 -->