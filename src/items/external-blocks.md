# External blocks
# 外部块

>[external-blocks.md](https://github.com/rust-lang/reference/blob/master/src/items/external-blocks.md)\
>commit: 18d7140a96d3d898ccded0cc32a16675681b0846 \
>本译文最后维护日期：2020-10-21

> **<sup>句法</sup>**\
> _ExternBlock_ :\
> &nbsp;&nbsp; `extern` [_Abi_]<sup>?</sup> `{`\
> &nbsp;&nbsp; &nbsp;&nbsp; [_InnerAttribute_]<sup>\*</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp; _ExternalItem_<sup>\*</sup>\
> &nbsp;&nbsp; `}`
>
> _ExternalItem_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> (\
> &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; [_MacroInvocationSemi_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | ( [_Visibility_]<sup>?</sup> ( _ExternalStaticItem_ | _ExternalFunctionItem_ ) )\
> &nbsp;&nbsp; )
>
> _ExternalStaticItem_ :\
> &nbsp;&nbsp; `static` `mut`<sup>?</sup> [IDENTIFIER] `:` [_Type_] `;`
>
> _ExternalFunctionItem_ :\
> &nbsp;&nbsp; `fn` [IDENTIFIER]&nbsp;[_Generics_]<sup>?</sup>\
> &nbsp;&nbsp; `(` ( _NamedFunctionParameters_ | _NamedFunctionParametersWithVariadics_ )<sup>?</sup> `)`\
> &nbsp;&nbsp; [_FunctionReturnType_]<sup>?</sup> [_WhereClause_]<sup>?</sup> `;`
>
> _NamedFunctionParameters_ :\
> &nbsp;&nbsp; _NamedFunctionParam_ ( `,` _NamedFunctionParam_ )<sup>\*</sup> `,`<sup>?</sup>
>
> _NamedFunctionParam_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> ( [IDENTIFIER] | `_` ) `:` [_Type_]
>
> _NamedFunctionParametersWithVariadics_ :\
> &nbsp;&nbsp; ( _NamedFunctionParam_ `,` )<sup>\*</sup> _NamedFunctionParam_ `,` [_OuterAttribute_]<sup>\*</sup> `...`

外部块提供未在当前 crate 中*定义*的数据项的*声明*，外部块是 Rust 外部函数接口的基础。这其实是某种类型的不受安全检查的导入。

外部块中允许两种形式的数据项*声明*：[函数][functions]和[静态项][statics]。只有在非安全(`unsafe`)上下文中才能调用在外部块中声明的函数或访问在外部块中声明的静态项。

## Functions
## 函数

外部块中的函数与其他 Rust 函数的声明方式相同，但这里中的函数可以没有主体，直接以分号结尾。外部块中的函数的参数不允许使用模式，只能使用[标识符/IDENTIFIER][IDENTIFIER]或 `_` 。

外部块中的函数可以被 Rust 代码调用，就像在 Rust 中定义的函数一样。Rust 编译器会自动在 Rust ABI 和外部 ABI 之间进行转换。

在外部块中声明的函数隐式为非安全(`unsafe`)的。当强转为函数指针时，外部块中声明的函数的类型就为 `unsafe extern "abi" for<'l1, ..., 'lm> fn(A1, ..., An) -> R`，其中 `'l1`，…`'lm` 是其生存期参数，`A1`，…，`An` 是该声明的参数的类型，`R` 是该声明的返回类型。

## Statics
## 静态项

[静态项][statics]在外部块内部与在外部块外部的声明方式相同，只是在外部块内部没有初始化其值的表达式。访问外部块中声明的静态项是 `unsafe` 的，不管它是否可变，因为无法保证静态项的内存位模式(bit pattern)对声明它的类型是有效的，因为初始化这些静态项的可能是任何外部代码（例如C）。

就像外部块之外的[静态项][statics]，外部静态项可以是不可变的，也可以是可变的。不可变外部静态项在执行任何 Rust 代码之前，*必须*被初始化。也就是说仅在 Rust 代码读取外部静态项之前对其进行初始化是不够的。

## ABI

不指定 ABI 字符串的默认情况下，外部块假设它们当前调用库时所使用的 ABI 约定是使用指定平台上的标准 C ABI。其他的 ABI 约定可以使用字符串 `abi` 指定，具体如下所示：

```rust
// 指定使用 stdcall 调用约定去调用 Windows API
extern "stdcall" { }
```

有三个 ABI 字符串是跨平台的，并且保证所有编译器都支持它们：

* `extern "Rust"` -- 在任何 Rust 语言中编写一个常用函数 `fn foo()` 时默认使用的 ABI。
* `extern "C"` -- 这等价于 `extern fn foo()`；无论您的 C编译器支持什么默认 ABI。
* `extern "system"` -- 通常等价于 `extern "C"`，除了在 Win32 平台上。在 Win32 平台上，应该使用`"stdcall"`，或者其他应该使用的 ABI 字符串来链接到 Windows API 自身。

还有一些特定于平台的 ABI 字符串：

* `extern "cdecl"` -- 默认为调用 x86\_32 C 所使用的调用约定。
* `extern "stdcall"` -- 默认为调用 x86\_32架构下的 Win32 API 所使用的调用约定 
* `extern "win64"` -- 默认为调用 x86\_64 Windows 平台下的 C 所使用调用约定。
* `extern "sysv64"` -- 默认为调用 非Windows x86\_64 平台下的 C 所使用的调用约定。
* `extern "aapcs"` -- 默认为调用 ARM 接口所使用的调用约定
* `extern "fastcall"` -- `fastcall` ABI——对应于 MSVC 的`__fastcall` 和 GCC 以及 clang 的 `__attribute__((fastcall))`。
* `extern "vectorcall"` -- `vectorcall` ABI ——对应于 MSVC 的 `__vectorcall` 和 clang 的 `__attribute__((vectorcall))`。

## Variadic functions
## 可变参数函数

可以在外部块内的函数的参数列表中的一个或多个命名参数后通过引入 `...` 来让该函数成为可变参数函数：

```rust
extern {
    fn foo(x: i32, ...);
}
```

## Attributes on extern blocks
## 外部块上的属性

下面列出的[属性][attributes]可以控制外部块的行为。

### The `link` attribute
### `link`属性

*`link`属性*为外部(`extern`)块中的数据项指定编译器应该链接的本机库的名称。它使用 [_MetaListNameValueStr_]元项属性句法规则指定其输入。`name`键指定要链接的本机库的名称。`kind`键是一个可选值，它指定具有以下可能值的库类型：

- `dylib` — 表示库类型是动态库。如果没有指定 `kind`，这是默认值。
- `static` — 表示库类型是静态库。
- `framework` — 表示库类型是 macOS 框架。这只对 macOS 目标平台有效。

如果指定了 `kind`，则必须指定 `name`键。

当从主机环境导入 symbols 时，`wasm_import_module`键可用于为外部(`extern`)块中的数据项指定 [WebAssembly模块][WebAssembly module]名称。如果未指定 `wasm_import_module`，则默认模块名为 `env`。

<!-- ignore: requires extern linking -->
```rust,ignore
#[link(name = "crypto")]
extern {
    // …
}

#[link(name = "CoreFoundation", kind = "framework")]
extern {
    // …
}

#[link(wasm_import_module = "foo")]
extern {
    // …
}
```

在空外部块上添加 `link`属性是有效的。可以用这种方式来满足代码中其他地方的外部块的链接需求(包括上游 crate)，而不必向每个外部块都添加此属性。

### The `link_name` attribute
### `link_name`属性


The `link_name` attribute may be specified on declarations inside an `extern`
block to indicate the symbol to import for the given function or static. It
uses the [_MetaNameValueStr_] syntax to specify the name of the symbol.
可以在外部(`extern`)块内的数据项声明上指定 `link_name`属性，可以用它来指示要为给定函数或静态项导入的具体 symbol。它使用 [_MetaNameValueStr_]元项属性句法规则指定 symbol 的名称。

```rust
extern {
    #[link_name = "actual_symbol_name"]
    fn name_in_rust();
}
```

### Attributes on function parameters
### 函数参数上的属性

外部函数参数上的属性遵循与[常规函数参数][regular function parameters]相同的规则和限制。

[IDENTIFIER]: ../identifiers.md
[WebAssembly module]: https://webassembly.github.io/spec/core/syntax/modules.html
[functions]: functions.md
[statics]: static-items.md
[_Abi_]: functions.md
[_FunctionReturnType_]: functions.md
[_Generics_]: generics.md
[_InnerAttribute_]: ../attributes.md
[_MacroInvocationSemi_]: ../macros.md#macro-invocation
[_MetaListNameValueStr_]: ../attributes.md#meta-item-attribute-syntax
[_MetaNameValueStr_]: ../attributes.md#meta-item-attribute-syntax
[_OuterAttribute_]: ../attributes.md
[_Type_]: ../types.md#type-expressions
[_Visibility_]: ../visibility-and-privacy.md
[_WhereClause_]: generics.md#where-clauses
[attributes]: ../attributes.md
[regular function parameters]: functions.md#attributes-on-function-parameters

<!-- 2020-10-16 -->
<!-- checked -->
