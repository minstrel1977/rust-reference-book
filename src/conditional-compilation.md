# 条件编译

>[conditional-compilation.md](https://github.com/rust-lang/reference/blob/master/src/conditional-compilation.md)\
>commit 5c857384ca17d0c81d48cc4bd0ae0d03c54580b7

> **<sup>句法</sup>**\
> _ConfigurationPredicate_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _ConfigurationOption_\
> &nbsp;&nbsp; | _ConfigurationAll_\
> &nbsp;&nbsp; | _ConfigurationAny_\
> &nbsp;&nbsp; | _ConfigurationNot_
>
> _ConfigurationOption_ :\
> &nbsp;&nbsp; [IDENTIFIER]&nbsp;(`=` ([STRING_LITERAL] | [RAW_STRING_LITERAL]))<sup>?</sup>
>
> _ConfigurationAll_\
> &nbsp;&nbsp; `all` `(` _ConfigurationPredicateList_<sup>?</sup> `)`
>
> _ConfigurationAny_\
> &nbsp;&nbsp; `any` `(` _ConfigurationPredicateList_<sup>?</sup> `)`
>
> _ConfigurationNot_\
> &nbsp;&nbsp; `not` `(` _ConfigurationPredicate_ `)`
>
> _ConfigurationPredicateList_\
> &nbsp;&nbsp; _ConfigurationPredicate_ (`,` _ConfigurationPredicate_)<sup>\*</sup> `,`<sup>?</sup>

根据某些条件， < !——这个定义有点空洞——> *条件编译的源代码*可能被认为是 crate 源代码的一部分，也可能不被认为是 crate 源代码的一部分。可以使用[attributes] [`cfg`] 和 [`cfg_attr`] 以及内置的 [`cfg` macro] 有条件地编译源代码。这些条件基于已编译的 crate 的目标架构、传递给编译器的任意值，以及下面将详细描述的其他一些事项。

每种形式的条件编译都有一个计算结果为真或假的*配置谓词*。谓词是以下内容之一：

* 一个配置选项。如果设置了该选项，则为真，如果未设置则为假。
* `all()` 这样的配置谓词列表，列表内的项以逗号分隔。如果至少有一个谓词为假，则为假。如果没有谓词，则为真。
* `any()` 这样的配置谓词列表，列表内的项以逗号分隔。如果至少有一个谓词为真，则为真。如果没有谓词，则为假。
* `not()` 配置谓词。如果它的谓词为假，整个配置它为真；如果它的谓词为真，整个配置为假。

*配置选项*是已设置或未设置的名称和键值对。名称以单个标识符形式写入，例如 `unix`。键值对被写为标识符，`=`，然后是字符串。例如，`target_arch=“x86_64”`是一个配置选项。

> **注意**: `=` 周围的空白将被忽略。`foo="bar"` 和 `foo = "bar"` 是等价的配置选项。

键在键-值配置选项集中不是唯一的。例如，`feature = "std"` and `feature = "serde"` 可以同时设置。

## 设置配置选项

设置哪些配置选项是在 crate 编译期间静态确定的。某些选项是根据相关编译的数据由*编译器设置集*设置的。其他选项是任意设置集，需从代码之外传参给编译器来主观设置。无法在正在编译的 crate 的源代码中设置编译配置选项。

> **注意**: 对于 `rustc`，使用 [`--cfg`] 标记设置任意配置集配置选项。

> **注意**: 带有关键字 `feature` 的配置选项是 [Cargo][cargo-feature] 用于指定编译时选项和可选依赖项的约定。

<div class="warning">

警告：任意设置的配置选项可能与编译器设置的配置选项具有相同的值。例如，在编译到 Windows 目标时，可以执行 `rustc --cfg "unix" program.rs` ，同时设置了 `unix` 和 `windows` 配置选项。实际上这样做是不明智的。

</div>

### `target_arch`

使用编译目标的 CPU 体系结构来设置一次键值选项。该值类似于平台的目标三元组（the platform's target triple）的第一个元素，但不相同。

示例值：

* `"x86"`
* `"x86_64"`
* `"mips"`
* `"powerpc"`
* `"powerpc64"`
* `"arm"`
* `"aarch64"`

### `target_feature`

可用于当前编译目标的各个平台特性相关的键-值选项设置。

示例值：

* `"avx"`
* `"avx2"`
* `"crt-static"`
* `"rdrand"`
* `"sse"`
* `"sse2"`
* `"sse4.1"`

有关可用特性的更多细节，请参见[`target_feature` 属性]。`target_feature` 选项提供了一个 `crt-static` 的附加特性，用于指示一个[静态 C 运行时]可用。

### `target_os`

Key-value option set once with the target's operating system. This value is
similar to the second and third element of the platform's target triple.

Example values:

* `"windows"`
* `"macos"`
* `"ios"`
* `"linux"`
* `"android"`
* `"freebsd"`
* `"dragonfly"`
* `"openbsd"`
* `"netbsd"`

### `target_family`

Key-value option set at most once with the target's operating system value.

Example values:

* `"unix"`
* `"windows"`

### `unix` and `windows`

`unix` is set if `target_family = "unix"` is set and `windows` is set if
`target_family = "windows"` is set.

### `target_env`

Key-value option set with further disambiguating information about the target
platform with information about the ABI or `libc` used. For historical reasons,
this value is only defined as not the empty-string when actually needed for
disambiguation. Thus, for example, on many GNU platforms, this value will be
empty. This value is similar to the fourth element of the platform's target
triple. One difference is that embedded ABIs such as `gnueabihf` will simply
define `target_env` as `"gnu"`.

Example values:

* `""`
* `"gnu"`
* `"msvc"`
* `"musl"`
* `"sgx"`

### `target_endian`

Key-value option set once with either a value of "little" or "big" depending
on the endianness of the target's CPU.

### `target_pointer_width`

Key-value option set once with the target's pointer width in bits.

Example values:

* `"16"`
* `"32"`
* `"64"`

### `target_vendor`

Key-value option set once with the vendor of the target.

Example values:

* `"apple"`
* `"fortanix"`
* `"pc"`
* `"unknown"`

### `test`

Enabled when compiling the test harness. Done with `rustc` by using the
[`--test`] flag. See [Testing] for more on testing support.

### `debug_assertions`

Enabled by default when compiling without optimizations.
This can be used to enable extra debugging code in development but not in
production.  For example, it controls the behavior of the standard library's
[`debug_assert!`] macro.

### `proc_macro`

Set when the crate being compiled is being compiled with the `proc_macro`
[crate type].

## Forms of conditional compilation

### The `cfg` attribute

> **<sup>Syntax</sup>**\
> _CfgAttrAttribute_ :\
> &nbsp;&nbsp; `cfg` `(` _ConfigurationPredicate_ `)`

<!-- should we say they're active attributes here? -->

The `cfg` [attribute] conditionally includes the thing it is attached to based
on a configuration predicate.

It is written as `cfg`, `(`, a configuration predicate, and finally `)`.

If the predicate is true, the thing is rewritten to not have the `cfg` attribute
on it. If the predicate is false, the thing is removed from the source code.

Some examples on functions:

```rust
// The function is only included in the build when compiling for macOS
#[cfg(target_os = "macos")]
fn macos_only() {
  // ...
}

// This function is only included when either foo or bar is defined
#[cfg(any(foo, bar))]
fn needs_foo_or_bar() {
  // ...
}

// This function is only included when compiling for a unixish OS with a 32-bit
// architecture
#[cfg(all(unix, target_pointer_width = "32"))]
fn on_32bit_unix() {
  // ...
}

// This function is only included when foo is not defined
#[cfg(not(foo))]
fn needs_not_foo() {
  // ...
}
```

The `cfg` attribute is allowed anywhere attributes are allowed.

### The `cfg_attr` attribute

> **<sup>Syntax</sup>**\
> _CfgAttrAttribute_ :\
> &nbsp;&nbsp; `cfg_attr` `(` _ConfigurationPredicate_ `,` _CfgAttrs_<sup>?</sup> `)`
>
> _CfgAttrs_ :\
> &nbsp;&nbsp; [_Attr_]&nbsp;(`,` [_Attr_])<sup>\*</sup> `,`<sup>?</sup>

The `cfg_attr` [attribute] conditionally includes [attributes] based on a
configuration predicate.

When the configuration predicate is true, this attribute expands out to the
attributes listed after the predicate. For example, the following module will
either be found at `linux.rs` or `windows.rs` based on the target.

<!-- ignore: `mod` needs multiple files -->
```rust,ignore
#[cfg_attr(target_os = "linux", path = "linux.rs")]
#[cfg_attr(windows, path = "windows.rs")]
mod os;
```

Zero, one, or more attributes may be listed. Multiple attributes will each be
expanded into separate attributes. For example:

<!-- ignore: fake attributes -->
```rust,ignore
#[cfg_attr(feature = "magic", sparkles, crackles)]
fn bewitched() {}

// When the `magic` feature flag is enabled, the above will expand to:
#[sparkles]
#[crackles]
fn bewitched() {}
```

> **Note**: The `cfg_attr` can expand to another `cfg_attr`. For example,
> `#[cfg_attr(target_os = "linux", cfg_attr(feature = "multithreaded", some_other_attribute))]`
> is valid. This example would be equivalent to
> `#[cfg_attr(all(target_os = "linux", feature ="multithreaded"), some_other_attribute)]`.

The `cfg_attr` attribute is allowed anywhere attributes are allowed.

### The `cfg` macro

The built-in `cfg` macro takes in a single configuration predicate and evaluates
to the `true` literal when the predicate is true and the `false` literal when
it is false.

For example:

```rust
let machine_kind = if cfg!(unix) {
  "unix"
} else if cfg!(windows) {
  "windows"
} else {
  "unknown"
};

println!("I'm running on a {} machine!", machine_kind);
```

[IDENTIFIER]: identifiers.md
[RAW_STRING_LITERAL]: tokens.md#raw-string-literals
[STRING_LITERAL]: tokens.md#string-literals
[Testing]: attributes/testing.md
[_Attr_]: attributes.md
[`--cfg`]: ../rustc/command-line-arguments.html#--cfg-configure-the-compilation-environment
[`--test`]: ../rustc/command-line-arguments.html#--test-build-a-test-harness
[`cfg`]: #the-cfg-attribute
[`cfg` macro]: #the-cfg-macro
[`cfg_attr`]: #the-cfg_attr-attribute
[`debug_assert!`]: ../std/macro.debug_assert.html
[`target_feature` attribute]: attributes/codegen.md#the-target_feature-attribute
[attribute]: attributes.md
[attributes]: attributes.md
[cargo-feature]: ../cargo/reference/features.html
[crate type]: linkage.md
[static C runtime]: linkage.md#static-and-dynamic-c-runtimes
