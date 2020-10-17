# 外部块

>[external-blocks.md](https://github.com/rust-lang/reference/blob/master/src/items/external-blocks.md)\
>commit: 18d7140a96d3d898ccded0cc32a16675681b0846

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

外部块提供未在当前 crate 中*定义*的数据项的*声明*；外部块是 Rust 外部函数接口的基础。这类似于未经检查的导入。

外部块中允许两种类型的数据项*声明*：[函数]和[静态项]。调用在外部块中声明的函数或访问在外部块中声明的静态项都只能在 `unsafe` 上下文中才被允许。

## 函数

外部块中的函数与其他 Rust 函数的声明方式相同，但这里中的函数可以没有主体，直接以分号结尾。外部块中的函数的参数不允许使用模式，只能使用[IDENTIFIER]或 `_` 。

外部块中的函数可以由 Rust 代码调用，就像在 Rust 中定义的函数一样。Rust 编译器会自动在Rust ABI 和外部ABI 之间进行转换。

在外部块中声明的函数隐式为 `unsafe`。当强制转换为函数指针时，外部块中声明的函数的类型就为 `unsafe extern "abi" for<'l1, ..., 'lm> fn(A1, ..., An) -> R`，其中 `'l1`，…`'lm` 是其生存期参数，`A1`，…，`An` 是该声明的参数的类型，`R` 是该声明的返回类型。

## 静态项

外部块内的静态项与外部块之外的[静态项]声明方式相同，只是这里没有初始化其值的表达式。访问外部块中声明的静态项是 `unsafe` 的，不管它是否可变，因为无法保证静态项的内存位模式对声明它的类型是有效的，因为初始化这些静态项的可能是任何外部代码（例如C）。

外部静态项可以是不可变的，也可以是可变的，就像外部块之外的[静态项]。在执行任何 Rust 代码之前，*必须*初始化不可变静态项。在 Rust 代码读取外部静态项之前对其进行初始化是不够的。

## ABI

默认情况下，外部块假设它们正在调用的库使用的是特定平台上的标准 C ABI。其他的 ABI 可以使用字符串 `abi` 指定，具体如下所示：

```rust
// 指向 Windows API 的接口
extern "stdcall" { }
```

有三个 ABI 字符串是跨平台的，并且保证所有编译器都支持它们：

* `extern "Rust"` -- 在任何 Rust 代码中编写一个正常的 `fn foo()` 时默认使用的 ABI。
 * `extern "C"` -- 这等价于 `extern fn foo()`；无论 C编译器支持什么默认值<!--whatever the default your C compiler supports. NeedRecheck-->。
* `extern "C"` -- 通常等价于 `extern "C"`，除了在 Win32 平台上。在 Win32 平台上，应该使用`"stdcall"` 或者其他应该使用的 ABI 字符串，来链接到 Windows API 自身。

还有一些特定于平台的 ABI 字符串：

* `extern "cdecl"` -- x86\_32 上 C代码的默认值。
* `extern "stdcall"` -- x86\_32 上 Win32 API 的默认值 
* `extern "win64"` -- x86\_64 Windows 上 C代码的默认值。
* `extern "sysv64"` -- 非Windows x86\_64 上的 C代码的默认值。
* `extern "aapcs"` -- ARM 的默认值
* `extern "fastcall"` -- `fastcall` ABI——对应于 MSVC 的`__fastcall` 和 GCC 以及 clang 的 `__attribute__((fastcall))`。
* `extern "vectorcall"` -- `vectorcall` ABI ——对应于 MSVC 的 `__vectorcall` 和 clang 的 `__attribute__((vectorcall))`。

## 可变参数函数

可以在外部块内的函数的参数列表中的一个或多个命名参数后通过指定 `...` 来让该函数成为可变参数函数：

```rust
extern {
    fn foo(x: i32, ...);
}
```

## 外部块上的属性

下面的[属性]可以控制外部块的行为。

### `link`属性

*`link`属性*为外部块中的数据项指定编译器应该链接的本机库的名称。它使用 [_MetaListNameValueStr_] 句法规则指定其输入。`name` 键指定要链接的本机库的名称。`kind` 键是一个可选值，它指定具有以下可能值的库类型：

- `dylib` — 动态库。如果没有指定 `kind`，这是默认值Indicates a dynamic library. This is the default if `kind` is not specified.
- `static` — 静态库。
- `framework` —  macOS 框架。这只对 macOS 目标平台有效。

如果指定了 `kind`，则必须包含 `name`键。

当从主机环境导入 symbols 时，键 `wasm_import_module` 可用于为 `extern`块中的项指定 [WebAssembly模块]名称。如果未指定 `wasm_import_module`，则默认模块名为`env`。

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

在空的外部块上添加 `link`属性是有效的。可以用这种方式来满足代码中其他地方的外部块的链接需求(包括上游 crate)，而不必向每个外部块都添加属性。

### `link_name`属性

`link_name`属性可以在 `extern`块内的声明中指定，它用来指示要为给定函数或静态项导入的具体 symbol。它使用 [_MetaNameValueStr_] 句法规则指定 symbol 的名称。

```rust
extern {
    #[link_name = "actual_symbol_name"]
    fn name_in_rust();
}
```

### 函数参数上的属性

外部函数参数上的属性遵循与[常规函数参数]相同的规则和限制。

[IDENTIFIER]: ../identifiers.md
[WebAssembly module]: https://webassembly.github.io/spec/core/syntax/modules.html
[函数]: functions.md
[静态项]: static-items.md
[_Abi_]: functions.md
[_FunctionReturnType_]: functions.md
[_Generics_]: generics.md
[_InnerAttribute_]: ../attributes.md
[_MacroInvocationSemi_]: ../macros.md#宏调用
[_MetaListNameValueStr_]: ../attributes.md#元项属性句法
[_MetaNameValueStr_]: ../attributes.md#元项属性句法
[_OuterAttribute_]: ../attributes.md
[_Type_]: ../types.md#type-expressions
[_Visibility_]: ../visibility-and-privacy.md
[_WhereClause_]: generics.md#where子句
[属性]: ../attributes.md
[常规函数参数]: functions.md#函数参数上的属性
