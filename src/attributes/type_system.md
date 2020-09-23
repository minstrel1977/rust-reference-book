# 类型系统属性

以下[属性]用于更改类型的使用方式。

## `non_exhaustive`属性

*`non_exhaustive`属性*表示类型或变体将来可能会添加更多字段或变体。它可以应用于[struct]、[enum] 和 枚举变体。

`non_exhaustive`属性使用 [_MetaWord_]句法规则，因此不接受任何输入。

在当前（`non_exhaustive`限制的数据项的）定义所在的 crate 内，`non_exhaustive` 没有效果。

```rust
#[non_exhaustive]
pub struct Config {
    pub window_width: u16,
    pub window_height: u16,
}

#[non_exhaustive]
pub enum Error {
    Message(String),
    Other,
}

pub enum Message {
    #[non_exhaustive] Send { from: u32, to: u32, contents: String },
    #[non_exhaustive] Reaction(u32),
    #[non_exhaustive] Quit,
}

// 非穷尽结构体可以在定义它的 crate 中正常构造。
let config = Config { window_width: 640, window_height: 480 };

// 非穷尽结构体可以在定义它的 crate 中进行详尽匹配
if let Config { window_width, window_height } = config {
    // ...
}

let error = Error::Other;
let message = Message::Reaction(3);

// 非穷尽枚举可以在定义它的 crate 中进行详尽匹配
match error {
    Error::Message(ref s) => { },
    Error::Other => { },
}

match message {
    // 非穷尽变体可以在定义它的 crate 中进行详尽匹配
    Message::Send { from, to, contents } => { },
    Message::Reaction(id) => { },
    Message::Quit => { },
}
```

在定义所在的 crate之外，注解为 `non_exhaustive` 的类型须在添加新字段或变体时保持向后兼容性。

非穷尽类型不能在定义它的 crate 之外构造：

- Non-exhaustive variants ([`struct`][struct] or [`enum` variant][enum]) cannot be constructed
  with a [_StructExpression_] \(including with [functional update syntax]).
- [`enum`][enum] instances can be constructed in an [_EnumerationVariantExpression_].

<!-- ignore: requires external crates -->
```rust,ignore
// `Config`, `Error`, and `Message` are types defined in an upstream crate that have been
// annotated as `#[non_exhaustive]`.
use upstream::{Config, Error, Message};

// Cannot construct an instance of `Config`, if new fields were added in
// a new version of `upstream` then this would fail to compile, so it is
// disallowed.
let config = Config { window_width: 640, window_height: 480 };

// Can construct an instance of `Error`, new variants being introduced would
// not result in this failing to compile.
let error = Error::Message("foo".to_string());

// Cannot construct an instance of `Message::Send` or `Message::Reaction`,
// if new fields were added in a new version of `upstream` then this would
// fail to compile, so it is disallowed.
let message = Message::Send { from: 0, to: 1, contents: "foo".to_string(), };
let message = Message::Reaction(0);

// Cannot construct an instance of `Message::Quit`, if this were converted to
// a tuple-variant `upstream` then this would fail to compile.
let message = Message::Quit;
```

There are limitations when matching on non-exhaustive types outside of the defining crate:

- When pattern matching on a non-exhaustive variant ([`struct`][struct] or [`enum` variant][enum]),
  a [_StructPattern_] must be used which must include a `..`. Tuple variant constructor visibility
  is lowered to `min($vis, pub(crate))`.
- When pattern matching on a non-exhaustive [`enum`][enum], matching on a variant does not
  contribute towards the exhaustiveness of the arms.

<!-- ignore: requires external crates -->
```rust, ignore
// `Config`, `Error`, and `Message` are types defined in an upstream crate that have been
// annotated as `#[non_exhaustive]`.
use upstream::{Config, Error, Message};

// Cannot match on a non-exhaustive enum without including a wildcard arm.
match error {
  Error::Message(ref s) => { },
  Error::Other => { },
  // would compile with: `_ => {},`
}

// Cannot match on a non-exhaustive struct without a wildcard.
if let Ok(Config { window_width, window_height }) = config {
    // would compile with: `..`
}

match message {
  // Cannot match on a non-exhaustive struct enum variant without including a wildcard.
  Message::Send { from, to, contents } => { },
  // Cannot match on a non-exhaustive tuple or unit enum variant.
  Message::Reaction(type) => { },
  Message::Quit => { },
}
```

Non-exhaustive types are always considered inhabited in downstream crates.

[_EnumerationVariantExpression_]: ../expressions/enum-variant-expr.md
[_MetaWord_]: ../attributes.md#meta-item-attribute-syntax
[_StructExpression_]: ../expressions/struct-expr.md
[_StructPattern_]: ../patterns.md#struct-patterns
[_TupleStructPattern_]: ../patterns.md#tuple-struct-patterns
[`if let`]: ../expressions/if-expr.md#if-let-expressions
[`match`]: ../expressions/match-expr.md
[attributes]: ../attributes.md
[enum]: ../items/enumerations.md
[functional update syntax]: ../expressions/struct-expr.md#functional-update-syntax
[struct]: ../items/structs.md
