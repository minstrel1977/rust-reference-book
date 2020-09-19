# 外部块

>[external-blocks.md](https://github.com/rust-lang/reference/blob/master/src/items/external-blocks.md)\
>commit 18d7140a96d3d898ccded0cc32a16675681b0846

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

在外部块中声明的函数隐式为 `unsafe`。当强制转换为函数指针时，外部块中声明的函数的类型就为 `unsafe extern "abi" for<'l1, ..., 'lm> fn(A1, ..., An) -> R`，其中 `'l1`，…`'lm` 是其生命周期参数，`A1`，…，`An` 是该声明的参数的类型，`R` 是该声明的返回类型。

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

* `extern "cdecl"` -- 为 x86\_32 C 代码准备的默认值。
* `extern "stdcall"` -- The default for the Win32 API on x86\_32.
* `extern "win64"` -- The default for C code on x86\_64 Windows.
* `extern "sysv64"` -- The default for C code on non-Windows x86\_64.
* `extern "aapcs"` -- The default for ARM.
* `extern "fastcall"` -- The `fastcall` ABI -- corresponds to MSVC's
  `__fastcall` and GCC and clang's `__attribute__((fastcall))`
* `extern "vectorcall"` -- The `vectorcall` ABI -- corresponds to MSVC's
  `__vectorcall` and clang's `__attribute__((vectorcall))`

## Variadic functions

Functions within external blocks may be variadic by specifying `...` after one
or more named arguments in the argument list:

```rust
extern {
    fn foo(x: i32, ...);
}
```

## Attributes on extern blocks

The following [attributes] control the behavior of external blocks.

### The `link` attribute

The *`link` attribute* specifies the name of a native library that the
compiler should link with for the items within an `extern` block. It uses the
[_MetaListNameValueStr_] syntax to specify its inputs. The `name` key is the
name of the native library to link. The `kind` key is an optional value which
specifies the kind of library with the following possible values:

- `dylib` — Indicates a dynamic library. This is the default if `kind` is not
  specified.
- `static` — Indicates a static library.
- `framework` — Indicates a macOS framework. This is only valid for macOS
  targets.

The `name` key must be included if `kind` is specified.

The `wasm_import_module` key may be used to specify the [WebAssembly module]
name for the items within an `extern` block when importing symbols from the
host environment. The default module name is `env` if `wasm_import_module` is
not specified.

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

It is valid to add the `link` attribute on an empty extern block. You can use
this to satisfy the linking requirements of extern blocks elsewhere in your
code (including upstream crates) instead of adding the attribute to each extern
block.

### The `link_name` attribute

The `link_name` attribute may be specified on declarations inside an `extern`
block to indicate the symbol to import for the given function or static. It
uses the [_MetaNameValueStr_] syntax to specify the name of the symbol.

```rust
extern {
    #[link_name = "actual_symbol_name"]
    fn name_in_rust();
}
```

### Attributes on function parameters

Attributes on extern function parameters follow the same rules and
restrictions as [regular function parameters].

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
