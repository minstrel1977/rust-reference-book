# 代码生成属性

>[codegen.md](https://github.com/rust-lang/reference/blob/master/src/attributes/codegen.md)\
>commit bc1a70875865bd578b473a431fb66788fb635886

下述[属性]用于控制代码生成。

## 优化提示

`cold` 和 `inline`[属性]给出了相关生成代码方式的建议，这种方法可能比没有提示时更快。属性只是提示，可能会被忽略。

这两个属性都可以在[函数]上使用。当应用于一个[trait]中的一个函数时，它们只应用于trait实现未覆盖此默认函数的情况，而不是所有trait实现。如果 trait 中，相关函数只有声明，没有实现，那这些属性对trait 函数没任何影响。

### `inline`属性

*`inline`[属性]*建议在调用者中放置带有此属性的函数的副本，而不是在定义函数的地方生成调用此函数的代码。

> ***注意***：`rustc` 编译器会根据启发式算法（internal heuristics，译者注：可字面理解为内部试探）自动内联函数。不正确的内联函数会使程序变慢，所以应该小心使用此属性。

使用 inline属性有三种方法：

* `#[inline]` *建议*执行内联扩展。
* `#[inline(always)]` *建议*应该一直执行内联扩展。
* `#[inline(never)]` *建议*应该从不执行内联扩展。

> ***注意***: `#[inline]` 在每种形式中都是一个提示，在 Rust 中不*需要*在调用者中放置带有此属性的函数的副本。

### `cold`属性

*`cold`[属性]*表示带有此属性的函数不太可能被调用。

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

标记为 `target_feature` 的函数不会内联到不支持给定特性的上下文中。`#[inline(always)]` 属性不能与 `target_feature`属性一起使用。

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
`sse4.1`    | `sse3`   | [SSE4.1] — Streaming SIMD Extensions 4.1 | 单指令多数据流扩展指令集4.1
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

### Behavior

将此属性应用到函数 `f` 上将允许 `f` 中的代码获得 `f` 被调用时建立的调用栈里“最顶层”的跟踪调用 [`Location`] 提示。从观察的角度来看，此属性的实现表现地就像从 `f` 的帧向上遍历堆栈，以找到*未分配*函数“outer”的最近帧，并返回“outer”中跟踪调用的[`Location`]。
Applying the attribute to a function `f` allows code within `f` to get a hint of the [`Location`] of the "topmost" tracked call that led to `f`'s invocation. At the point of observation, an implementation behaves as if it walks up the stack from `f`'s frame to find the nearest frame of an *unattributed* function `outer`, and it returns the [`Location`] of the tracked call in `outer`.

```rust
#[track_caller]
fn f() {
    println!("{}", std::panic::Location::caller());
}
```

> 注意：`core` 提供 [`core::panic::Location::caller`] 来观察调用者的位置。它封装了由 `rustc` 实现的内部函数 [`core::intrinsics::caller_location`]。

> 注意：由于结果 `Location` 是一个提示，所以实现可能会提前停止向堆栈的遍历。（[Limitations]#注意事项]限制。
> 注意:由于结果的“位置”是一个提示，实现可能会提前停止它在堆栈上的遍历。请参阅[Limitations](# Limitations)以获得重要的注意事项。
> Note: because the resulting `Location` is a hint, an implementation may halt its walk up the stack early. See [Limitations](#limitations) for important caveats.

#### Examples

When `f` is called directly by `calls_f`, code in `f` observes its callsite within `calls_f`:

```rust
# #[track_caller]
# fn f() {
#     println!("{}", std::panic::Location::caller());
# }
fn calls_f() {
    f(); // <-- f() prints this location
}
```

When `f` is called by another attributed function `g` which is in turn called by `calls_g`, code in both `f` and `g` observes `g`'s callsite within `calls_g`: 

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
    g(); // <-- g() prints this location twice, once itself and once from f()
}
```

When `g` is called by another attributed function `h` which is in turn called by `calls_h`, all code in `f`, `g`, and `h` observes `h`'s callsite within `calls_h`:

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
    h(); // <-- prints this location three times, once itself, once from g(), once from f()
}
```

And so on.

### Limitations

This information is a hint and implementations are not required to preserve it.

In particular, coercing a function with `#[track_caller]` to a function pointer creates a shim which appears to observers to have been called at the attributed function's definition site, losing actual caller information across virtual calls. A common example of this coercion is the creation of a trait object whose methods are attributed.

> Note: The aforementioned shim for function pointers is necessary because `rustc` implements `track_caller` in a codegen context by appending an implicit parameter to the function ABI, but this would be unsound for an indirect call because the parameter is not a part of the function's type and a given function pointer type may or may not refer to a function with the attribute. The creation of a shim hides the implicit parameter from callers of the function pointer, preserving soundness.

[_MetaListNameValueStr_]: ../attributes.md#meta-item-attribute-syntax
[`-C target-cpu`]: https://doc.rust-lang.org/rustc/codegen-options/index.html#target-cpu
[`-C target-feature`]: https://doc.rust-lang.org/rustc/codegen-options/index.html#target-feature
[`is_x86_feature_detected`]: https://doc.rust-lang.org/std/macro.is_x86_feature_detected.html
[`target_feature`-条件编译选项]: ../conditional-compilation.md#target_feature
[attribute]: ../attributes.md
[attributes]: ../attributes.md
[functions]: ../items/functions.md
[target architecture]: ../conditional-compilation.md#target_arch
[trait]: ../items/traits.md
[undefined behavior]: ../behavior-considered-undefined.md
[unsafe function]: ../unsafe-functions.md
[rust-abi]: ../items/external-blocks.md#abi
[`core::intrinsics::caller_location`]: https://doc.rust-lang.org/core/intrinsics/fn.caller_location.html
[`core::panic::Location::caller`]: https://doc.rust-lang.org/core/panic/struct.Location.html#method.caller
[`Location`]: https://doc.rust-lang.org/core/panic/struct.Location.html
