# 诊断属性

>[diagnostics.md](https://github.com/rust-lang/reference/blob/master/src/attributes/diagnostics.md)\
>commit 2196589ccf8fefc4edeaa0ab430fe2ce6a57dec3

以下[属性]用于在编译期间控制或生成诊断消息。

## lint检查类属性

lint检查(lint check，译者注：这是一个名词)命名了一些潜在的不合规的编码模式，例如执行不到的代码（unreachable-code）或未提供文档（missing_docs）。lint属性（`allow`、`warn`、`deny` 和 `forbid`）通过使用[_MetaListPaths_]句法格式指定 lint检查的名称列表的方式来更改应用该属性的实体的 lint级别。

对任何 lint检查 `C`（译者注：这个 `C` 不是C语言的C，应该是原作者取 check 的首字母的大写来代表某一 lint检查的意思）：

* `allow(C)` 会无视对 `C` 的检查，那这样的违规行为就不会被报告，
* `warn(C)` 警告违反 `C` ，但继续编译。
* `deny(C)` 遇到违反 `C` 的情况会触发错误（signals an error），
* `forbid(C)` 与 `deny(C)` 相同，但同时会禁止以后再更改 lint级别，
* 
> 注意：可以通过 `rustc -W help` 找到所有 `rustc` 支持的 lint检查，以及它们的默认设置。也可以在[rustc book]中找到相关文档。

```rust
pub mod m1 {
    // 这里忽略未提供文档的编码行为
    #[allow(missing_docs)]
    pub fn undocumented_one() -> i32 { 1 }

    // 未提供文档的编码行为在这里将触发一个告警
    #[warn(missing_docs)]
    pub fn undocumented_too() -> i32 { 2 }

    // 未提供文档的编码行为在这里将触发一个错误
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

        // 尽管上面允许，但未提供文档的编码行为在这里将触发一个告警
        #[warn(missing_docs)]
        pub fn undocumented_two() -> i32 { 2 }
    }

    // 未提供文档的编码行为在这里将触发一个告警
    pub fn undocumented_too() -> i32 { 3 }
}
```

此示例展示了如何使用 `forbid` 来禁止对该 lint检查 使用 `allow`：
This example shows how one can use `forbid` to disallow uses of `allow` for that lint check:

```rust,compile_fail
#[forbid(missing_docs)]
pub mod m3 {
    // 试图从 error级别切换到 allow级别
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

## The `deprecated` attribute

The *`deprecated` attribute* marks an item as deprecated. `rustc` will issue
warnings on usage of `#[deprecated]` items. `rustdoc` will show item
deprecation, including the `since` version and `note`, if available.

The `deprecated` attribute has several forms:

- `deprecated` — Issues a generic message.
- `deprecated = "message"` — Includes the given string in the deprecation
  message.
- [_MetaListNameValueStr_] syntax with two optional fields:
  - `since` — Specifies a version number when the item was deprecated. `rustc`
    does not currently interpret the string, but external tools like [Clippy]
    may check the validity of the value.
  - `note` — Specifies a string that should be included in the deprecation
    message. This is typically used to provide an explanation about the
    deprecation and preferred alternatives.

The `deprecated` attribute may be applied to any [item], [trait item], [enum
variant], [struct field], [external block item], or [macro definition]. It
cannot be applied to [trait implementation items]. When applied to an item
containing other items, such as a [module] or [implementation], all child
items inherit the deprecation attribute.
<!-- NOTE: It is only rejected for trait impl items
(AnnotationKind::Prohibited). In all other locations, it is silently ignored.
Tuple struct fields are ignored.
-->

Here is an example:

```rust
#[deprecated(since = "5.2", note = "foo was rarely used. Users should instead use bar")]
pub fn foo() {}

pub fn bar() {}
```

The [RFC][1270-deprecation.md] contains motivations and more details.

[1270-deprecation.md]: https://github.com/rust-lang/rfcs/blob/master/text/1270-deprecation.md

## The `must_use` attribute

The *`must_use` attribute* is used to issue a diagnostic warning when a value
is not "used". It can be applied to user-defined composite types
([`struct`s][struct], [`enum`s][enum], and [`union`s][union]), [functions],
and [traits].

The `must_use` attribute may include a message by using the
[_MetaNameValueStr_] syntax such as `#[must_use = "example message"]`. The
message will be given alongside the warning.

When used on user-defined composite types, if the [expression] of an
[expression statement] has that type, then the `unused_must_use` lint is
violated.

```rust
#[must_use]
struct MustUse {
    // some fields
}

# impl MustUse {
#   fn new() -> MustUse { MustUse {} }
# }
#
// Violates the `unused_must_use` lint.
MustUse::new();
```

When used on a function, if the [expression] of an [expression statement] is a
[call expression] to that function, then the `unused_must_use` lint is
violated.

```rust
#[must_use]
fn five() -> i32 { 5i32 }

// Violates the unused_must_use lint.
five();
```

When used on a [trait declaration], a [call expression] of an [expression
statement] to a function that returns an [impl trait] of that trait violates
the `unused_must_use` lint.

```rust
#[must_use]
trait Critical {}
impl Critical for i32 {}

fn get_critical() -> impl Critical {
    4i32
}

// Violates the `unused_must_use` lint.
get_critical();
```

When used on a function in a trait declaration, then the behavior also applies
when the call expression is a function from an implementation of the trait.

```rust
trait Trait {
    #[must_use]
    fn use_me(&self) -> i32;
}

impl Trait for i32 {
    fn use_me(&self) -> i32 { 0i32 }
}

// Violates the `unused_must_use` lint.
5i32.use_me();
```

When used on a function in a trait implementation, the attribute does nothing.

> Note: Trivial no-op expressions containing the value will not violate the
> lint. Examples include wrapping the value in a type that does not implement
> [`Drop`] and then not using that type and being the final expression of a
> [block expression] that is not used.
>
> ```rust
> #[must_use]
> fn five() -> i32 { 5i32 }
>
> // None of these violate the unused_must_use lint.
> (five(),);
> Some(five());
> { five() };
> if true { five() } else { 0i32 };
> match true {
>     _ => five()
> };
> ```

> Note: It is idiomatic to use a [let statement] with a pattern of `_`
> when a must-used value is purposely discarded.
>
> ```rust
> #[must_use]
> fn five() -> i32 { 5i32 }
>
> // Does not violate the unused_must_use lint.
> let _ = five();
> ```

[Clippy]: https://github.com/rust-lang/rust-clippy
[_MetaListNameValueStr_]: ../attributes.md#meta-item-attribute-syntax
[_MetaListPaths_]: ../attributes.md#meta-item-attribute-syntax
[_MetaNameValueStr_]: ../attributes.md#meta-item-attribute-syntax
[`Drop`]: ../special-types-and-traits.md#drop
[attributes]: ../attributes.md
[block expression]: ../expressions/block-expr.md
[call expression]: ../expressions/call-expr.md
[enum variant]: ../items/enumerations.md
[enum]: ../items/enumerations.md
[expression statement]: ../statements.md#expression-statements
[expression]: ../expressions.md
[external block item]: ../items/external-blocks.md
[functions]: ../items/functions.md
[impl trait]: ../types/impl-trait.md
[implementation]: ../items/implementations.md
[item]: ../items.md
[let statement]: ../statements.md#let-statements
[macro definition]: ../macros-by-example.md
[module]: ../items/modules.md
[rustc book]: https://doc.rust-lang.org/rustc/lints/index.html
[struct field]: ../items/structs.md
[struct]: ../items/structs.md
[trait declaration]: ../items/traits.md
[trait implementation items]: ../items/implementations.md#trait-implementations
[trait item]: ../items/traits.md
[traits]: ../items/traits.md
[union]: ../items/unions.md
