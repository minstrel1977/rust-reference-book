{{#include attributes-redirect.html}}
# 属性

>[attributes.md](https://github.com/rust-lang/reference/blob/master/src/attributes.md)\
>commit 814a530db0a3f91821095a830fa321fdd5a41d17

> **<sup>句法</sup>**\
> _InnerAttribute_ :\
> &nbsp;&nbsp; `#` `!` `[` _Attr_ `]`
>
> _OuterAttribute_ :\
> &nbsp;&nbsp; `#` `[` _Attr_ `]`
>
> _Attr_ :\
> &nbsp;&nbsp; [_SimplePath_] _AttrInput_<sup>?</sup>
>
> _AttrInput_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_DelimTokenTree_]\
> &nbsp;&nbsp; | `=` [_LiteralExpression_]<sub>_without suffix_</sub>

*属性*是一种通用的、形式自由的元数据，这种元数据会被（编译器）依据名称、约定、语言和编译器版本进行解释。（Rust 的）属性是根据 [ECMA-335]标准中的属性规范进行建模的，其语法来自 [ECMA-334] \(C#）。

*内部属性*以 `#!` 开头的方式编写，应用于它在其中声明的数据项。*外部属性*以不后跟感叹号的(`!`)的 `#` 开头的方式编写，应用于属性后面的内容。

属性由指向属性的路径，和路径后跟的可选的带定界符的标记树（其解释由属性定义）组成。除了宏属性之外，其他属性也允许输入是等号(`=`)后跟文字表达式的格式。更多细节请参见下面的[元数据项句法](#元数据项属性句法)。

属性可以分为以下几类：

* [内置属性]
* [哄属性][attribute macros]
* [派生宏辅助属性]
* [外部工具属性](#外部工具属性)

属性可以应用于语言中的许多事物：

* 所有的[数据项声明]都接受外部属性，同时[外部块]、[函数]、[实现]和[模块]也还接受内部属性。
* 大多数[语句]接受外部属性(参见[表达式属性]，了解表达式语句的限制)。
* [块表达式]接受外部和内部属性，但只有当它们是另一个[表达式语句]的外部表达式时或时另一个块表达式的最终表达式(final expression)时才能接受。
* [枚举]变体和[结构体]、[联合体]的字段接受外部属性。
* [匹配表达式的匹配臂][match expressions]接受外部属性。
* [泛型生命周期或类型参数][generics]接受外部属性。
* 表达式在受限情况下接受外部属性，详见[表达式属性]。
* [函数][functions]、[闭包]和[函数指针]的参数接受外部属性。这包括函数指针和[外部块]中用 `...` 表示的可变参数上的属性。

属性的一些例子：

```rust
// 应用于封闭模块或 crate 的普通元数据。
#![crate_type = "lib"]

// 标记为单元测试的函数
#[test]
fn test_foo() {
    /* ... */
}

// 条件编译模块
#[cfg(target_os = "linux")]
mod bar {
    /* ... */
}

// 用于取消警告/错误的 lint属性
#[allow(non_camel_case_types)]
type int8_t = i8;

// 适用于整个函数的内部属性
fn some_unused_variables() {
  #![allow(unused_variables)]

  let x = ();
  let y = ();
  let z = ();
}
```

## 元数据项属性句法

“元数据项”是大多数[内置属性]应用 Attr句法规则(见本章头部的句法格式)的句法。它有以下语法格式：

> **<sup>句法</sup>**\
> _MetaItem_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_SimplePath_]\
> &nbsp;&nbsp; | [_SimplePath_] `=` [_LiteralExpression_]<sub>_without suffix_</sub>\
> &nbsp;&nbsp; | [_SimplePath_] `(` _MetaSeq_<sup>?</sup> `)`
>
> _MetaSeq_ :\
> &nbsp;&nbsp; _MetaItemInner_ ( `,` MetaItemInner )<sup>\*</sup> `,`<sup>?</sup>
>
> _MetaItemInner_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _MetaItem_\
> &nbsp;&nbsp; | [_LiteralExpression_]<sub>_without suffix_</sub>

元数据项中的文字表达式不能包含整数或浮点类型的后缀。

各种内置属性使用元数据项语法的不同子集来指定它们的输入。下面的语法规则展示了一些常用的使用形式：

> **<sup>句法</sup>**\
> _MetaWord_:\
> &nbsp;&nbsp; [IDENTIFIER]
>
> _MetaNameValueStr_:\
> &nbsp;&nbsp; [IDENTIFIER] `=` ([STRING_LITERAL] | [RAW_STRING_LITERAL])
>
> _MetaListPaths_:\
> &nbsp;&nbsp; [IDENTIFIER] `(` ( [_SimplePath_] (`,` [_SimplePath_])* `,`<sup>?</sup> )<sup>?</sup> `)`
>
> _MetaListIdents_:\
> &nbsp;&nbsp; [IDENTIFIER] `(` ( [IDENTIFIER] (`,` [IDENTIFIER])* `,`<sup>?</sup> )<sup>?</sup> `)`
>
> _MetaListNameValueStr_:\
> &nbsp;&nbsp; [IDENTIFIER] `(` ( _MetaNameValueStr_ (`,` _MetaNameValueStr_)* `,`<sup>?</sup> )<sup>?</sup> `)`

元数据项的一些例子是：

形式 | 示例
------|--------
_MetaWord_ | `no_std`
_MetaNameValueStr_ | `doc = "example"`
_MetaListPaths_ | `allow(unused, clippy::inline_always)`
_MetaListIdents_ | `macro_use(foo, bar)`
_MetaListNameValueStr_ | `link(name = "CoreFoundation", kind = "framework")`

## 活动属性和惰性属性

属性要么是活动的，要么是惰性的。在属性处理过程中，*活动属性*将自己从它们所在的对象中移除，而*惰性属性*保持不变。

[`cfg`] 和 [`cfg_attr`] 属性是活动的。[`test`] 属性在为测试所做的编译中是惰性的，否则是活动的。[属性宏]是活动的。所有其他属性都是惰性的。

## 外部工具属性

编译器可能允许外部工具的属性，其中每个工具都驻留在自己的命名空间中。属性路径的第一段是工具的名称，再加上一个或多个工具自己解释的附加段。

当工具未被使用时，该工具的属性将被接受而没有警告。在使用该工具时，该工具负责处理和解释其属性。

如果使用了 [`no_implicit_prelude`] 属性，则外部工具属性不可用。

```rust
// 告诉rustfmt工具不要格式化以下元素。
#[rustfmt::skip]
struct S {
}

// 控制clippy工具的“圈复杂度(cyclomatic complexity)”阈值。
#[clippy::cyclomatic_complexity = "100"]
pub fn f() {}
```

> 注意: `rustc` 目前能识别的工具是 “clippy” 和 “rustfmt”。

## 内置属性的索引表

下面是所有内置属性的索引：
The following is an index of all built-in attributes.

- Conditional compilation
  - [`cfg`] — Controls conditional compilation.
  - [`cfg_attr`] — Conditionally includes attributes.
- Testing
  - [`test`] — Marks a function as a test.
  - [`ignore`] — Disables a test function.
  - [`should_panic`] — Indicates a test should generate a panic.
- Derive
  - [`derive`] — Automatic trait implementations.
  - [`automatically_derived`] — Marker for implementations created by
    `derive`.
- Macros
  - [`macro_export`] — Exports a `macro_rules` macro for cross-crate usage.
  - [`macro_use`] — Expands macro visibility, or imports macros from other
    crates.
  - [`proc_macro`] — Defines a function-like macro.
  - [`proc_macro_derive`] — Defines a derive macro.
  - [`proc_macro_attribute`] — Defines an attribute macro.
- Diagnostics
  - [`allow`], [`warn`], [`deny`], [`forbid`] — Alters the default lint level.
  - [`deprecated`] — Generates deprecation notices.
  - [`must_use`] — Generates a lint for unused values.
- ABI, linking, symbols, and FFI
  - [`link`] — Specifies a native library to link with an `extern` block.
  - [`link_name`] — Specifies the name of the symbol for functions or statics
    in an `extern` block.
  - [`no_link`] — Prevents linking an extern crate.
  - [`repr`] — Controls type layout.
  - [`crate_type`] — Specifies the type of crate (library, executable, etc.).
  - [`no_main`] — Disables emitting the `main` symbol.
  - [`export_name`] — Specifies the exported symbol name for a function or
    static.
  - [`link_section`] — Specifies the section of an object file to use for a
    function or static.
  - [`no_mangle`] — Disables symbol name encoding.
  - [`used`] — Forces the compiler to keep a static item in the output
    object file.
  - [`crate_name`] — Specifies the crate name.
- Code generation
  - [`inline`] — Hint to inline code.
  - [`cold`] — Hint that a function is unlikely to be called.
  - [`no_builtins`] — Disables use of certain built-in functions.
  - [`target_feature`] — Configure platform-specific code generation.
  - [`track_caller`] - Pass the parent call location to `std::panic::Location::caller()`.
- Documentation
  - `doc` — Specifies documentation. See [The Rustdoc Book] for more
    information. [Doc comments] are transformed into `doc` attributes.
- Preludes
  - [`no_std`] — Removes std from the prelude.
  - [`no_implicit_prelude`] — Disables prelude lookups within a module.
- Modules
  - [`path`] — Specifies the filename for a module.
- Limits
  - [`recursion_limit`] — Sets the maximum recursion limit for certain
    compile-time operations.
  - [`type_length_limit`] — Sets the maximum size of a polymorphic type.
- Runtime
  - [`panic_handler`] — Sets the function to handle panics.
  - [`global_allocator`] — Sets the global memory allocator.
  - [`windows_subsystem`] — Specifies the windows subsystem to link with.
- Features
  - `feature` — Used to enable unstable or experimental compiler features. See
    [The Unstable Book] for features implemented in `rustc`.
- Type System
  - [`non_exhaustive`] — Indicate that a type will have more fields/variants
    added in future.

[Doc comments]: comments.md#doc-comments
[ECMA-334]: https://www.ecma-international.org/publications/standards/Ecma-334.htm
[ECMA-335]: https://www.ecma-international.org/publications/standards/Ecma-335.htm
[Expression Attributes]: expressions.md#expression-attributes
[IDENTIFIER]: identifiers.md
[RAW_STRING_LITERAL]: tokens.md#raw-string-literals
[STRING_LITERAL]: tokens.md#string-literals
[The Rustdoc Book]: ../rustdoc/the-doc-attribute.html
[The Unstable Book]: ../unstable-book/index.html
[_DelimTokenTree_]: macros.md
[_LiteralExpression_]: expressions/literal-expr.md
[_SimplePath_]: paths.md#simple-paths
[`allow`]: attributes/diagnostics.md#lint-check-attributes
[`automatically_derived`]: attributes/derive.md#the-automatically_derived-attribute
[`cfg_attr`]: conditional-compilation.md#the-cfg_attr-attribute
[`cfg`]: conditional-compilation.md#the-cfg-attribute
[`cold`]: attributes/codegen.md#the-cold-attribute
[`crate_name`]: crates-and-source-files.md#the-crate_name-attribute
[`crate_type`]: linkage.md
[`deny`]: attributes/diagnostics.md#lint-check-attributes
[`deprecated`]: attributes/diagnostics.md#the-deprecated-attribute
[`derive`]: attributes/derive.md
[`export_name`]: abi.md#the-export_name-attribute
[`forbid`]: attributes/diagnostics.md#lint-check-attributes
[`global_allocator`]: runtime.md#the-global_allocator-attribute
[`ignore`]: attributes/testing.md#the-ignore-attribute
[`inline`]: attributes/codegen.md#the-inline-attribute
[`link_name`]: items/external-blocks.md#the-link_name-attribute
[`link_section`]: abi.md#the-link_section-attribute
[`link`]: items/external-blocks.md#the-link-attribute
[`macro_export`]: macros-by-example.md#path-based-scope
[`macro_use`]: macros-by-example.md#the-macro_use-attribute
[`must_use`]: attributes/diagnostics.md#the-must_use-attribute
[`no_builtins`]: attributes/codegen.md#the-no_builtins-attribute
[`no_implicit_prelude`]: items/modules.md#prelude-items
[`no_link`]: items/extern-crates.md#the-no_link-attribute
[`no_main`]: crates-and-source-files.md#the-no_main-attribute
[`no_mangle`]: abi.md#the-no_mangle-attribute
[`no_std`]: crates-and-source-files.md#preludes-and-no_std
[`non_exhaustive`]: attributes/type_system.md#the-non_exhaustive-attribute
[`panic_handler`]: runtime.md#the-panic_handler-attribute
[`path`]: items/modules.md#the-path-attribute
[`proc_macro_attribute`]: procedural-macros.md#attribute-macros
[`proc_macro_derive`]: procedural-macros.md#derive-macros
[`proc_macro`]: procedural-macros.md#function-like-procedural-macros
[`recursion_limit`]: attributes/limits.md#the-recursion_limit-attribute
[`repr`]: type-layout.md#representations
[`should_panic`]: attributes/testing.md#the-should_panic-attribute
[`target_feature`]: attributes/codegen.md#the-target_feature-attribute
[`test`]: attributes/testing.md#the-test-attribute
[`track_caller`]: attributes/codegen.md#the-track_caller-attribute
[`type_length_limit`]: attributes/limits.md#the-type_length_limit-attribute
[`used`]: abi.md#the-used-attribute
[`warn`]: attributes/diagnostics.md#lint-check-attributes
[`windows_subsystem`]: runtime.md#the-windows_subsystem-attribute
[attribute macros]: procedural-macros.md#attribute-macros
[block expressions]: expressions/block-expr.md
[built-in attributes]: #built-in-attributes-index
[derive macro helper attributes]: procedural-macros.md#derive-macro-helper-attributes
[enum]: items/enumerations.md
[expression statement]: statements.md#expression-statements
[external blocks]: items/external-blocks.md
[functions]: items/functions.md
[generics]: items/generics.md
[implementations]: items/implementations.md
[item declarations]: items.md
[match expressions]: expressions/match-expr.md
[modules]: items/modules.md
[statements]: statements.md
[struct]: items/structs.md
[union]: items/unions.md
[closure]: expressions/closure-expr.md
[function pointer]: types/function-pointer.md
[variadic functions]: items/external-blocks.html#variadic-functions
