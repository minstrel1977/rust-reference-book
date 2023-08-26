# External blocks
# 外部块

>[external-blocks.md](https://github.com/rust-lang/reference/blob/master/src/items/external-blocks.md)\
>commit: f0bb14c9bacca5d305e0488c43273ebe22fe928e \
>本章译文最后维护日期：2023-08-26

> **<sup>句法</sup>**\
> _ExternBlock_ :\
> &nbsp;&nbsp; `unsafe`<sup>?</sup> `extern` [_Abi_]<sup>?</sup> `{`\
> &nbsp;&nbsp; &nbsp;&nbsp; [_InnerAttribute_]<sup>\*</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp; _ExternalItem_<sup>\*</sup>\
> &nbsp;&nbsp; `}`
>
> _ExternalItem_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> (\
> &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; [_MacroInvocationSemi_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | ( [_Visibility_]<sup>?</sup> ( [_StaticItem_] | [_Function_] ) )\
> &nbsp;&nbsp; )

外部块提供未在当前 crate 中*定义*的程序项的*声明*，外部块是 Rust 外部函数接口的基础。这其实是某种意义上的不受安全检查的导入入口。

外部块里允许存在两种形式的程序项*声明*：[函数][functions]和[静态项][statics]。只有在非安全(`unsafe`)上下文中才能调用在外部块中声明的函数或访问在外部块中声明的静态项。

在句法上，关键字 `unsafe` 允许出现在关键字 `extern` 之前，但是在语义层面却会被弃用。这种设计允许宏在将关键字 `unsafe` 从 token流中移除之前利用此句法来使用此关键字。

## Functions
## 函数

外部块中的函数与其他 Rust函数的声明方式相同，但这里的函数不能有函数体，取而代之的是直接以分号结尾。外部块中的函数的参数不允许使用模式，只能使用[标识符(IDENTIFIER)][IDENTIFIER] 或 `_` 。函数限定符（`const`、`async`、`unsafe` 和 `extern`）也不允许在这里使用。

外部块中的函数可以被 Rust 代码调用，就跟调用在 Rust 中定义的函数一样。Rust 编译器会自动在 Rust ABI 和外部 ABI 之间进行转换。

在外部块中声明的函数隐式为非安全(`unsafe`)的。当强转为函数指针时，外部块中声明的函数的类型就为 `unsafe extern "abi" for<'l1, ..., 'lm> fn(A1, ..., An) -> R`，其中 `'l1`，…`'lm` 是其生存期参数，`A1`，…，`An` 是该声明的参数的类型，`R` 是该声明的返回类型。

## Statics
## 静态项

[静态项][statics]在外部块内部与在外部块之外的声明方式相同，只是在外部块内部声明的静态项没有对应的初始化表达式。访问外部块中声明的静态项是 `unsafe` 的，不管它是否可变，因为没有任何保证来确保静态项的内存位模式(bit pattern)对声明它的类型是有效的，因为初始化这些静态项的可能是其他的任意外部代码（例如 C）。

就像外部块之外的[静态项][statics]，外部静态项可以是不可变的，也可以是可变的。在执行任何 Rust 代码之前，不可变外部静态项*必须*被初始化。也就是说，对于外部静态项，仅在 Rust 代码读取它之前对它进行初始化是不够的。

## ABI

不指定 ABI 字符串的默认情况下，外部块会假定使用指定平台上的标准 C ABI 约定来调用当前的库。其他的 ABI 约定可以使用字符串 `abi` 来指定，具体如下所示：

```rust
// 到 Windows API 的接口。（译者注：指定使用 stdcall调用约定去调用 Windows API）
extern "stdcall" { }
```

有三个 ABI 字符串是跨平台的，并且保证所有编译器都支持它们：

* `extern "Rust"` -- 在任何 Rust 语言中编写的普通函数 `fn foo()` 默认使用的 ABI。
* `extern "C"` -- 这等价于 `extern fn foo()`；无论您的 C编译器支持什么默认 ABI。
* `extern "system"` -- 在 Win32 平台之外，中通常等价于 `extern "C"`。在 Win32 平台上，应该使用`"stdcall"`，或者其他应该使用的 ABI 字符串来链接它们自身的 Windows API。

还有一些特定于平台的 ABI 字符串：

* `extern "cdecl"` -- 通过 FFI 调用 x86\_32 C 资源所使用的默认调用约定。
* `extern "stdcall"` -- 通过 FFI 调用 x86\_32架构下的 Win32 API 所使用的默认调用约定 
* `extern "win64"` -- 通过 FFI 调用 x86\_64 Windows 平台下的 C 资源所使用的默认调用约定。
* `extern "sysv64"` -- 通过 FFI 调用 非Windows x86\_64 平台下的 C 资源所使用的默认调用约定。
* `extern "aapcs"` --通过 FFI 调用 ARM 接口所使用的默认调用约定
* `extern "fastcall"` -- `fastcall` ABI——对应于 MSVC 的`__fastcall` 和 GCC 以及 clang 的 `__attribute__((fastcall))`。
* `extern "vectorcall"` -- `vectorcall` ABI ——对应于 MSVC 的 `__vectorcall` 和 clang 的 `__attribute__((vectorcall))`。
* `extern "thiscall"` -- MSVC 下调用 C++ 成员函数的默认约定 -- 对应与 MSVC 下的`__thiscall`，以及 GCC 和 clang 的`__attribute__((thiscall))` 调用约定
* `extern "efiapi"` -- 调用 [UEFI] 函数所使用的 ABI。

## Variadic functions
## 可变参数函数

可以在外部块内的函数的参数列表中的一个或多个具名参数后通过引入 `...` 来让该函数成为可变参数函数。注意可变参数`...` 前至少有一个具名参数，并且只能位于参数列表的最后。可变参数可以通过标识符来指定：

```rust
extern "C" {
    fn foo(x: i32, ...);
    fn with_name(format: *const u8, args: ...);
}
```

## Attributes on extern blocks
## 外部块上的属性

下面列出的[属性][attributes]可以控制外部块的行为。

### The `link` attribute
### `link`属性

*`link`属性*为外部(`extern`)块中的程序项指定编译器应该链接的本地库的名称。它使用 [_MetaListNameValueStr_]元项属性句法指定其输入参数。`name`键指定要链接的本地库的名称。`kind`键是一个可选值，它指定具有以下可选值的库类型：

- `dylib` — 表示要链接的库类型是动态库。如果没有指定 `kind`，这是默认值。
- `static` — 表示要链接的库类型是静态库。
- `framework` — 表示要链接的库类型是 macOS 框架。这只对 macOS 目标平台有效。
- `raw-dylib` — 表示要链接的库类型是动态库，但具体要链接到哪个动态库，链接器会使用本次编译出的库作为导入库来定位识别链接，（相关详细信息，请参阅下面的[`dylib` vs `rawdylib`][`dylib` versus `raw-dylib`]）。此属性仅对 Windows目标平台有效。

如果指定了 `kind`键，则必须有 `name`键。

可选的 `modifiers`参数提供了一种为库指定链接修饰符的方法。
修饰符被指定为以逗号分隔的字符串，每个修饰符的前缀都是 `+` 或 `-`，分别表示修饰符处于启用或禁用状态。
当前不支持在单个 `link`属性中指定多个 `modifiers`参数，或在同一个 `modifiers`参数中指定多个相同的修饰符。

例如: `#[link(name = "mylib", kind = "static", modifiers = "+whole-archive")`.

当从主机环境导入 symbols 时，`wasm_import_module`键可用于为外部(`extern`)块中的程序项指定 [WebAssembly模块][WebAssembly module]名称。如果未指定 `wasm_import_module`，则默认模块名为 `env`。

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

在空外部块上添加 `link`属性是有效的。可以用这种方式来满足代码中其他地方的外部块的链接需求（包括上游 crate），而不必向每个外部块都添加此属性。

#### Linking modifiers: `bundle`
#### 链接修饰符: `bundle`

这个修饰符只在静态链接模式下适用。
在其他链接模式下会导致编译错误。

在构建 rlib 或 staticlib 时，`+bundle` 意味着本地静态库将被打包到 rlib 或者 staticlib类型的归档文件中，之后最终的二进制文件在在链接时，将从此处来检索。

当构建 rlib时，`-bundle` 意味着本机静态库会被“按名称”注册为该 rlib 的依赖项，并且其中的对象文件仅在最终的二进制文件的链接过程中才被包含进来，其中最终的链接过程会执行按该名称的文件搜索来定位那些对象文件。\
当构建 staticlib时，`-bundle`意味着本机静态库根本不会被包括在归档文件中，一些层次更高或则说是排序更靠后的构建系统会在之后的链接最终二进制文件的过程中才来添加它。

在构建其他目标（如可执行文件或动态库）时，此修饰符无效。

此修饰符的默认形式是 `+bundle`。

有关此修饰符的更多实现细节，请参见[rustc文档中的 `bundle`][`bundle` documentation for rustc]。

#### Linking modifiers: `whole-archive`
#### 链接修饰符: `whole-archive`

此修饰符仅兼容静态(`static`)链接类型。
使用任何其他链接类型都会引起编译器报错。

`+whole-archive` 意味着静态库会被链接为一个完整的 archive 文件，而不丢弃任何对象文件。

此修饰符的默认配置是 `-whole-archive`。

有关此修饰符的更多实现细节，请参见[rustc文档中的 `whole-archive`][`whole-archive` documentation for rustc]。

### Linking modifiers: `verbatim`
### 链接修饰符: `verbatim`

此修饰符兼容任何链接类型。

`+verbatim` 意味着 rustc 本身不会在库名称中添加任何目标平台指定的库前缀或后缀（如 `lib` 或 `.a`），并且会尽力向链接器请求相同的内容。

`-verbatim` 意味着 rustc 在将库名称传递给链接器之前，将在库名称中添加特定于目标平台的前缀和后缀，或者不会阻止链接器隐式添加它。

此修饰符的默认值为 `-verbatim`。

关于这个修饰符的更多实现细节可以在 rustc 的[`verbatim`文档][`verbatim` documentation for rustc] 中找到。

#### `dylib` versus `raw-dylib`
#### `dylib` vs `raw-dylib`

在 Windows 上，链接动态库需要先向链接器提供一个导入库：这是一个特殊的静态库，它声明了此动态库导出的所有符号，这样链接器就知道它们必须在运行时动态加载。

指定 `kind = "dylib"` 将指示 Rust编译器根据 `name`键链接导入库。然后，链接器将使用其正常的库解析逻辑来查找导入库。或者，使用 `kind = "raw-dylib"` 来指示编译器在编译期间生成一个导入库，并将其提供给链接器。

`raw dylib` 仅在 Windows 上受支持。在针对其他平台时使用它将导致编译器错误。

#### The `import_name_type` key
#### `import_name_type`键

在 x86 Windows上，函数的名称是“被修饰过的”（比如被添加了特定的前缀和/或后缀），以指示其支持的调用约定。例如，名为 `fn1` 且没有参数的 `stdcall`调用约定函数将被修饰为`_fn1@0`。然而，[PE格式][PE Format]也允许名称没有前缀或不加修饰。此外，MSVC和GNU工具链对相同的调用约定使用不同的装饰，这意味着，默认情况下，一些 Win32函数不能通过 GNU工具链使用 `raw-dylib`链接类型进行调用。

为了照顾到这些差异，在使用 `raw-dylib`链接类型时，你可以通过指定 `import_name_type`键使用下面的（某一）值来更改这些函数在生成的库文件中的命名方式：

* `decorated`：函数名称将使用 MSVC工具链格式进行完全修饰。
* `noprefix`：函数名称将使用 MSVC工具链格式进行修饰，但跳过前导的 `?`、`@` 或者可选地 `_`。
* `undecorated`：函数名称将不会被修饰。

如果未指定 `import_name_type`键，则函数名称将使用目标工具链的格式进行完全修饰。

变量从不会被修饰，因此 `import_name_type`键对它们在生成的库文件的命名方式没有影响。

`import_name_type`键仅在x86 Windows上受支持。在针对其他平台时使用它将导致编译器错误。

### The `link_name` attribute
### `link_name`属性

可以在外部(`extern`)块内的程序项声明上指定 *`link_name`属性*，可以用它来指示要为给定函数或静态项导入的具体 symbol。它使用 [_MetaNameValueStr_]元项属性句法指定 symbol 的名称。

```rust
extern {
    #[link_name = "actual_symbol_name"]
    fn name_in_rust();
}
```
此属性和 `link_ordinal`属性同时使用会导致编译器报错。

### The `link_ordinal` attribute
### `link_ordinal`属性

*`link_ordinal`属性*可以应用于外部(`extern`)块内的各种声明上，用以给当前编译生成的链接导入库在链接时要使用的数字序号。Windows 上的动态库导出的每个符号都有的唯一编号，当加载库时可以使用相应序号查找对应的符号，而不必按名称查找。

<div class="warning">

警告：`link_ordinal` 只能在此符号的序号已稳定了的情况下使用：如果在包含某符号的二进制文件构建时，此符号对应的序号未明确设置，则此构建将自动为其分配一个序号，并且在后续的库构建时此序号可能会在二进制文件生成之间还会发生变化。

</div>

<!-- ignore: Only works on x86 Windows -->
```rust,ignore
#[link(name = "exporter", kind = "raw-dylib")]
extern "stdcall" {
    #[link_ordinal(15)]
    fn imported_function_stdcall(i: i32);
}
```

此属性仅用于 `raw-dylib`链接类型。
使用任何其他类型都会导致编译器报错。

此属性和 `link_name`属性同时使用会导致编译器报错。

### Attributes on function parameters
### 函数参数上的属性

外部函数参数上的属性遵循与[常规函数参数][regular function parameters]相同的规则和限制。

[IDENTIFIER]: ../identifiers.md
[UEFI]: https://uefi.org/specifications
[WebAssembly module]: https://webassembly.github.io/spec/core/syntax/modules.html
[functions]: functions.md
[statics]: static-items.md
[_Abi_]: functions.md
[_Function_]: functions.md
[_InnerAttribute_]: ../attributes.md
[_MacroInvocationSemi_]: ../macros.md#macro-invocation
[_MetaListNameValueStr_]: ../attributes.md#meta-item-attribute-syntax
[_MetaNameValueStr_]: ../attributes.md#meta-item-attribute-syntax
[_OuterAttribute_]: ../attributes.md
[_StaticItem_]: static-items.md
[_Visibility_]: ../visibility-and-privacy.md
[attributes]: ../attributes.md
[regular function parameters]: functions.md#attributes-on-function-parameters
[`bundle` documentation for rustc]: https://doc.rust-lang.org/rustc/command-line-arguments.html#linking-modifiers-bundle
[`whole-archive` documentation for rustc]: https://doc.rust-lang.org/rustc/command-line-arguments.html#linking-modifiers-whole-archive
[`verbatim` documentation for rustc]: https://doc.rust-lang.org/rustc/command-line-arguments.html#linking-modifiers-verbatim
[`dylib` versus `raw-dylib`]: #dylib-versus-raw-dylib
[PE Format]: https://learn.microsoft.com/windows/win32/debug/pe-format#import-name-type