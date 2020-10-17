{{#include attributes-redirect.html}}
# 属性

>[attributes.md](https://github.com/rust-lang/reference/blob/master/src/attributes.md)\
>commit: 814a530db0a3f91821095a830fa321fdd5a41d17

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
* [宏属性][attribute macros]
* [派生宏辅助属性]
* [外部工具属性](#外部工具属性)

属性可以应用于语言中的许多事物：

* 所有的[数据项声明]都接受外部属性，同时[外部块]、[函数]、[实现]和[模块]也还接受内部属性。
* 大多数[语句]接受外部属性(参见[表达式属性]，了解表达式语句的限制)。
* [块表达式]接受外部和内部属性，但只有当它们是另一个[表达式语句]的外部表达式时或时另一个块表达式的最终表达式(final expression)时才能接受。
* [枚举]变体和[结构体]、[联合体]的字段接受外部属性。
* [匹配表达式的匹配臂][match expressions]接受外部属性。
* [泛型生存期或类型参数][generics]接受外部属性。
* 表达式在受限情况下接受外部属性，详见[表达式属性]。
* [函数][functions]、[闭包]和[函数指针]的参数接受外部属性。这包括函数指针和[外部块][variadic functions]中用 `...` 表示的可变参数上的属性。

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

## 元项属性句法

“元数据项”是大多数[内置属性]遵循 Attr句法规则(见本章头部的句法规则)的句法。它有以下语法格式：

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

[`cfg`] 和 [`cfg_attr`] 属性是活动的。[`test`] 属性在为测试所做的编译中是惰性的，否则是活动的。[宏属性][Attribute macros]是活动的。所有其他属性都是惰性的。

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

下面是所有内置属性的索引表：

- 条件编译
  - [`cfg`] — 控制条件编译。
  - [`cfg_attr`] — 有条件地包含属性。
- 测试
  - [`test`] — 将函数标记为测试函数。
  - [`ignore`] — 禁用测试功能。
  - [`should_panic`] — 表示测试应该产生 panic。
- 派生
  - [`derive`] — 自动 trait实现
  - [`automatically_derived`] — 由 `derive` 创建的实现标记。
- 宏
  - [`macro_export`] — 导出 `macro_rules` 宏用于跨 crate 的使用。
  - [`macro_use`] — 扩展宏可见性，或从其他 crate 导入宏。
  - [`proc_macro`] — 定义类函数宏。
  - [`proc_macro_derive`] — 定义派生宏。
  - [`proc_macro_attribute`] — 定义属性宏。
- 诊断
  - [`allow`]、[`warn`]、[`deny`]、[`forbid`] — 更改默认的 lint 级别。
  - [`deprecated`] — 生成弃用通知。
  - [`must_use`] — 为未使用的值生成 lint提醒。
- ABI、链接、symbol、和 FFI
  - [`link`] — 指定要与 `extern`块链接的本地库。
  - [`link_name`] — 指定 `extern`块中函数或静态项的 symbol名称。
  - [`no_link`] — 防止链接外部crate。
  - [`repr`] — 控制类型的布局。
  - [`crate_type`] — 指定 crate 的类型(库、可执行文件等)。
  - [`no_main`] — 禁止发布 `main` symbol。
  - [`export_name`] — 指定函数或静态项的导出 symbol名。
  - [`link_section`] — 指定用于函数或静态项的对象文件的部分。
  - [`no_mangle`] — 禁用 symbol名编码。
  - [`used`] — 强制编译器在输出对象文件中保留静态项。
  - [`crate_name`] — 指定 crate名。
- 代码生成
  - [`inline`] — 内联代码提示。
  - [`cold`] — 提示函数不太可能被调用。
  - [`no_builtins`] — 禁用某些内置函数的使用。
  - [`target_feature`] — 配置特定于平台的代码生成。
  - [`track_caller`] - 将父调用位置传递给 `std::panic::Location::caller()`。
- 文档
  - `doc` — 指定文档。更多信息见 [The Rustdoc Book]。[Doc注释]被转换为 `doc` 属性。
- 预导入模块集
  - [`no_std`] — 从预导入模块集中移除 std。
  - [`no_implicit_prelude`] — 在模块中禁用预导入模块查找。
- 模块
  - [`path`] — 指定模块的文件名。
- 限制
  - [`recursion_limit`] — 设置某些编译时操作的最大递归限制。
  - [`type_length_limit`] — 设置多态类型的最大尺寸。
- 运行时
  - [`panic_handler`] — 设置函数来处理 panic。
  - [`global_allocator`] — 设置全局内存分配器。
  - [`windows_subsystem`] — 指定要链接的windows子系统。
- 特性
  - `feature` — 用于启用非稳定的或实验性的编译器特性。参见 [The Unstable Book] 了解在 `rustc` 中实现的特性。
- 类型系统
  - [`non_exhaustive`] — 指示一个类型将来会添加更多的字段/变体。

[Doc注释]: comments.md#注释
[ECMA-334]: https://www.ecma-international.org/publications/standards/Ecma-334.htm
[ECMA-335]: https://www.ecma-international.org/publications/standards/Ecma-335.htm
[表达式属性]: expressions.md#表达式属性
[IDENTIFIER]: identifiers.md
[RAW_STRING_LITERAL]: tokens.md#raw-string-literals
[STRING_LITERAL]: tokens.md#string-literals
[The Rustdoc Book]: https://doc.rust-lang.org/nightly/rustdoc/
[The Unstable Book]: https://doc.rust-lang.org/nightly/unstable-book/
[_DelimTokenTree_]: macros.md
[_LiteralExpression_]: expressions/literal-expr.md
[_SimplePath_]: paths.md#简单路径
[`allow`]: attributes/diagnostics.md#lint检查类属性
[`automatically_derived`]: attributes/derive.md#automatically_derived属性
[`cfg_attr`]: conditional-compilation.md#cfg_attr属性
[`cfg`]: conditional-compilation.md#cfg属性
[`cold`]: attributes/codegen.md#cold属性
[`crate_name`]: crates-and-source-files.md#crate_name属性
[`crate_type`]: linkage.md
[`deny`]: attributes/diagnostics.md#lint检查类属性
[`deprecated`]: attributes/diagnostics.md#deprecated属性
[`derive`]: attributes/derive.md
[`export_name`]: abi.md#the-export_name-attribute
[`forbid`]: attributes/diagnostics.md#lint检查类属性
[`global_allocator`]: runtime.md#the-global_allocator-attribute
[`ignore`]: attributes/testing.md#ignore属性
[`inline`]: attributes/codegen.md#inline属性
[`link_name`]: items/external-blocks.md#link_name属性
[`link_section`]: abi.md#the-link_section-attribute
[`link`]: items/external-blocks.md#link属性
[`macro_export`]: macros-by-example.md#基于路径的作用域
[`macro_use`]: macros-by-example.md#macro_use属性
[`must_use`]: attributes/diagnostics.md#must_use属性
[`no_builtins`]: attributes/codegen.md#no_builtins属性
[`no_implicit_prelude`]: items/modules.md#预导入项
[`no_link`]: items/extern-crates.md#`no_link`属性
[`no_main`]: crates-and-source-files.md#no_main属性
[`no_mangle`]: abi.md#the-no_mangle-attribute
[`no_std`]: crates-and-source-files.md#预导入包和-no_std
[`non_exhaustive`]: attributes/type_system.md#the-non_exhaustive-attribute
[`panic_handler`]: runtime.md#the-panic_handler-attribute
[`path`]: items/modules.md#path属性
[`proc_macro_attribute`]: procedural-macros.md#属性宏
[`proc_macro_derive`]: procedural-macros.md#派生宏
[`proc_macro`]: procedural-macros.md#类函数过程宏
[`recursion_limit`]: attributes/limits.md#recursion_limit属性
[`repr`]: type-layout.md#representations
[`should_panic`]: attributes/testing.md#should_panic属性
[`target_feature`]: attributes/codegen.md#target_feature属性
[`test`]: attributes/testing.md#test属性
[`track_caller`]: attributes/codegen.md#track_caller属性
[`type_length_limit`]: attributes/limits.md#type_length_limit属性
[`used`]: abi.md#the-used-attribute
[`warn`]: attributes/diagnostics.md#lint检查类属性
[`windows_subsystem`]: runtime.md#the-windows_subsystem-attribute
[attribute macros]: procedural-macros.md#属性宏
[块表达式]: expressions/block-expr.md
[内置属性]: #内置属性的索引表
[派生宏辅助属性]: procedural-macros.md#派生宏辅助属性
[枚举]: items/enumerations.md
[表达式语句]: statements.md#表达式语句
[外部块]: items/external-blocks.md
[函数]: items/functions.md
[泛型]: items/generics.md
[实现]: items/implementations.md
[数据项声明]: items.md
[match expressions]: expressions/match-expr.md
[模块]: items/modules.md
[语句]: statements.md
[结构体]: items/structs.md
[联合体: items/unions.md
[闭包]: expressions/closure-expr.md
[函数指针]: types/function-pointer.md
[variadic functions]: items/external-blocks.md#可变参数函数
