# 诊断属性

>[diagnostics.md](https://github.com/rust-lang/reference/blob/master/src/attributes/diagnostics.md)\
>commit 2196589ccf8fefc4edeaa0ab430fe2ce6a57dec3

以下[属性]用于在编译期间控制或生成诊断消息。

## lint检查类属性

lint检查(lint check，译者注：这是一个名词)命名了一些潜在的不合规的编码模式，例如执行不到的代码（unreachable-code）或未提供文档（missing_docs）。lint属性（`allow`、`warn`、`deny` 和 `forbid`）通过使用[_MetaListPaths_]句法规则指定 lint检查的名称列表的方式来更改应用该属性的实体的 lint级别。

对任何 lint检查 `C`（译者注：这个 `C` 不是C语言的C，应该是原作者取 check 的首字母的大写来代表某一 lint检查的名字的意思）：

* `allow(C)` 会无视对 `C` 的检查，那这样的违规行为就不会被报告，
* `warn(C)` 警告违反 `C` ，但继续编译。
* `deny(C)` 遇到违反 `C` 的情况会触发编译器报错（signals an error），
* `forbid(C)` 与 `deny(C)` 相同，但同时会禁止以后再更改 lint级别，
* 
> 注意：可以通过 `rustc -W help` 找到所有 `rustc` 支持的 lint检查，以及它们的默认设置。也可以在[rustc book]中找到相关文档。

```rust
pub mod m1 {
    // 这里忽略未提供文档的编码行为
    #[allow(missing_docs)]
    pub fn undocumented_one() -> i32 { 1 }

    // 未提供文档的编码行为在这里将触发编译器告警
    #[warn(missing_docs)]
    pub fn undocumented_too() -> i32 { 2 }

    // 未提供文档的编码行为在这里将触发编译器报错
    #[deny(missing_docs)]
    pub fn undocumented_end() -> i32 { 3 }
}
```

此示例展示了如何使用 `allow` 和 `warn` 来打开和关闭一个特定的 lint检查：

```rust
#[warn(missing_docs)]
pub mod m2{
    #[allow(missing_docs)]
    pub mod nested {
        // 这里忽略未提供文档的编码行为
        pub fn undocumented_one() -> i32 { 1 }

        // 尽管上面允许，但未提供文档的编码行为在这里将触发编译器告警
        #[warn(missing_docs)]
        pub fn undocumented_two() -> i32 { 2 }
    }

    // 未提供文档的编码行为在这里将触发编译器告警
    pub fn undocumented_too() -> i32 { 3 }
}
```

此示例展示了如何使用 `forbid` 来禁止对该 lint检查 使用 `allow`：

```rust,compile_fail
#[forbid(missing_docs)]
pub mod m3 {
    // 试图切换到 allow级别将触发编译器报错
    #[allow(missing_docs)]
    /// 返回 2。
    pub fn undocumented_too() -> i32 { 2 }
}
```

### 工具类lint属性

工具类lint 允许为 `allow`、`warn`、`deny` 或 `forbid` 使用当前作用域内可用的特定的外部工具的lint。

目前，`clippy` 是唯一可用的 lint工具。

工具类lint 只有在相关工具处于活动状态时才会做对应的代码模式检查。如果一个 lint属性，如 `allow`，引用了一个不存在的工具类lint，编译器将不会警告不存在的 lint，除非您使用该工具。

否则，他们工作就像常规的 lint属性：

```rust
// 将整个 `pedantic` clippy lint组设置为告警级别
#![warn(clippy::pedantic)]
// 使来自 `filter_map` clippy lint 的警告静音
#![allow(clippy::filter_map)]

fn main() {
    // ...
}

// 使 `cmp_nan` clippy lint 静音，仅用于此函数
#[allow(clippy::cmp_nan)]
fn foo() {
    // ...
}
```

## `deprecated`属性

*`deprecated`属性*将数据项标记为已弃用。`rustc` 将对使用了被 `#[deprecated]` 限定的数据项的行为发出警告。`rustdoc` 将显示已弃用数据项的弃用信息，包括 `since` 版本和 `note` 提示（如果可用）。

`deprecated`属性有几种形式：

- `deprecated` — 发出一个通用的消息。
- `deprecated = "message"` — 在弃用消息中包含给定的字符串。
- [_MetaListNameValueStr_]句法里带有两个可选字段：
  - `since` — 指定数据项被弃用时的版本号。`rustc` 目前不解释此字符串，但是像 [Clippy] 这样的外部工具可以检查值的有效性。
  - `note` — 指定一个应该包含在弃用消息中的字符串。这通常用于提供关于不推荐的解释和推荐首选替代。

`deprecated`属性可以应用于任何[数据项]，[trait的内部项]，[枚举变体]，[结构体字段]，[外部块的内部项]，或[宏定义]。它不能应用于 [trait实现的内部项]。当应用于包含其他数据项的数据项时，如[模块]或[实现]，所有子数据项都继承 deprecation属性。

<!-- 注意: 它只被 trait实现(AnnotationKind::Prohibited)拒绝。在这些之外的所有其他位置，它会被静默忽略。应用于元组结构体的字段时，此属性直接被忽略。-->

这有个例子：

```rust
#[deprecated(since = "5.2", note = "foo was rarely used. Users should instead use bar")]
pub fn foo() {}

pub fn bar() {}
```

[RFC][1270-deprecation.md] 内有动机说明和更多细节。

[1270-deprecation.md]: https://github.com/rust-lang/rfcs/blob/master/text/1270-deprecation.md

## `must_use`属性

*`must_use`属性* 用于在值未被“使用”时发出诊断警告。它可以应用于用户定义的复合类型([`struct`s][结构体]、[`enum`s][枚举] 和 [`union`s][联合体])、[函数]和 [trait]。

`must_use`属性可以包含使用[_MetaNameValueStr_]句法规则的消息，如 `#[must_use = "example message"]`。该字符串将出现在警告信息里。

在用户定义的复合类型上使用时，如果[表达式语句]里的[表达式]具有该类型，那么 `unused_must_use` 这个lint（检查）就被违反了。

```rust
#[must_use]
struct MustUse {
    // 一些字段
}

# impl MustUse {
#   fn new() -> MustUse { MustUse {} }
# }
#
// 违反 `unused_must_use` lint。
MustUse::new();
```

当在函数上使用时，如果[表达式语句]的[表达式]是该函数的[调用表达式]，那么 `unused_must_use` lint 就被违反

```rust
#[must_use]
fn five() -> i32 { 5i32 }

// 违反 unused_must_use lint。
five();
```

在 [trait声明]中使用时，如果[表达式语句]的[调用表达式]返回了 trait 的 [impl trait] ，则违反了 `unused_must_use` lint。

```rust
#[must_use]
trait Critical {}
impl Critical for i32 {}

fn get_critical() -> impl Critical {
    4i32
}

// 违反 `unused_must_use` lint。
get_critical();
```

当在 trait声明中的函数上使用时，同时调用表达式是 trait实现中的函数时，该行为也适用。

```rust
trait Trait {
    #[must_use]
    fn use_me(&self) -> i32;
}

impl Trait for i32 {
    fn use_me(&self) -> i32 { 0i32 }
}

// 违反 `unused_must_use` lint。
5i32.use_me();
```

当在 trait实现中的函数上使用时，该属性将被忽略。

> 注意：包含值的普通无操作表达式不会违反该 lint。例如，将值包装在没有实现 [`Drop`] 的类型中，然后不使用该类型，并成为未使用的[块表达式]的最后一个表达式。
>
> ```rust
> #[must_use]
> fn five() -> i32 { 5i32 }
>
> // 这些都不违反 unused_must_use lint。
> (five(),);
> Some(five());
> { five() };
> if true { five() } else { 0i32 };
> match true {
>     _ => five()
> };
> ```

> 注意：当一个必须使用的值被故意丢弃时，使用带有模式 `_`的[let语句]是惯用的方法。
>
> ```rust
> #[must_use]
> fn five() -> i32 { 5i32 }
>
> // 不违反 unused_must_use lint。
> let _ = five();
> ```

[Clippy]: https://github.com/rust-lang/rust-clippy
[_MetaListNameValueStr_]: ../attributes.md#元项属性句法
[_MetaListPaths_]: ../attributes.md#元项属性句法
[_MetaNameValueStr_]: ../attributes.md#元项属性句法
[`Drop`]: ../special-types-and-traits.md#drop
[属性]: ../attributes.md
[块表达式]: ../expressions/block-expr.md
[调用表达式]: ../expressions/call-expr.md
[枚举变体]: ../items/enumerations.md
[枚举]: ../items/enumerations.md
[表达式语句]: ../statements.md#表达式语句
[表达式]: ../expressions.md
[外部块的内部项]: ../items/external-blocks.md
[函数]: ../items/functions.md
[impl trait]: ../types/impl-trait.md
[实现]: ../items/implementations.md
[数据项]: ../items.md
[let语句]: ../statements.md#let语句
[宏定义]: ../macros-by-example.md
[模块]: ../items/modules.md
[rustc book]: https://doc.rust-lang.org/rustc/lints/index.html
[结构体字段]: ../items/structs.md
[结构体]: ../items/structs.md
[trait声明]: ../items/traits.md
[trait实现的内部项]: ../items/implementations.md#trait实现
[trait的内部项]: ../items/traits.md
[trait]: ../items/traits.md
[联合体]: ../items/unions.md