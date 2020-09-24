# 类型系统属性

以下[属性]用于更改类型的使用方式。

## `non_exhaustive`属性

*`non_exhaustive`属性*表示类型或变体将来可能会添加更多字段或变体。它可以应用于[struct]、[enum] 和 枚举变体。

`non_exhaustive`属性使用 [_MetaWord_]句法规则，因此不接受任何输入。

在当前（`non_exhaustive`限制的类型）定义所在的 crate 内，`non_exhaustive` 没有效果。

```rust
#[non_exhaustive]
pub struct Config {
    pub window_width: u16,
    pub window_height: u16,
}

#[non_exhaustive]
pub enum Error {
    Message(String), // 译者注：此变体为元组变体
    Other,
}

pub enum Message {
    #[non_exhaustive] Send { from: u32, to: u32, contents: String },
    #[non_exhaustive] Reaction(u32),  // 译者注：此字段为元组变体型字段
    #[non_exhaustive] Quit, // 译者注：此字段添加变体后可成元组变体型字段
}

// 非穷尽结构体可以在定义它的 crate 中正常构建。
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

在定义所在的 crate之外，标注为 `non_exhaustive` 的类型须在添加新字段或变体时保持向后兼容性。

非穷尽类型不能在定义它的 crate 之外构建：

- 非穷尽变体（[结构体]或[枚举变体]）不能用 [_StructExpression_]句法格式（包括[函数更新句法]）构建。
- [枚举]示例能用[_EnumerationVariantExpression_]句法规则构建。

示例：（译者注：可以把上例看成本例的 `upstream` ）
<!-- ignore: requires external crates -->
```rust,ignore
// `Config`、`Error` `Message`是在上游 crate 中定义的类型，这些类型已被标注为 `#[non_exhaustive]`。
use upstream::{Config, Error, Message};

// 不能构造 `Config` 的实例，如果在 `upstream` 的新版本中添加了新字段，则本地编译会失败，因此不允许这样做。
let config = Config { window_width: 640, window_height: 480 };

// 可以构造 `Error` 的实例，引入的新变体不会导致编译失败。
let error = Error::Message("foo".to_string());

// 无法构造 `Message::Send` 或 `Message::Reaction` 的实例，
// 如果在 `upstream` 的新版本中添加了新字段，则本地编译失败，因此不允许。
let message = Message::Send { from: 0, to: 1, contents: "foo".to_string(), };
let message = Message::Reaction(0);

// 无法构造 `Message::Quit` 的实例，
// 如果 `upstream` 内的 `Message::Quit` 的因为添加字段变成元组变体（tuple-variant/tuple variant）后，则本地编译失败。
let message = Message::Quit;
```

在定义所在的 crate之外对非穷尽类型进行匹配，有如下限制:

- 当模式匹配一个非穷尽变体([结构体]或[枚举变体])时，必须使用 [_StructPattern_]句法规则进行，其匹配臂必须有一个为 `..`。元组变体的构造函数的可见性降低为 `min($vis, pub(crate))`。
- 当模式匹配在一个非穷尽的[枚举]上时，单个变体上的匹配不影响整个枚举的匹配详尽要求。

示例：（译者注：可以把上上例看成本例的 `upstream` ）
<!-- ignore: requires external crates -->
```rust, ignore
// `Config`、`Error` `Message`是在上游 crate 中定义的类型，这些类型已被标注为 `#[non_exhaustive]`。
use upstream::{Config, Error, Message};

// 不包含通配符匹配臂，无法匹配非穷尽枚举。
match error {
  Error::Message(ref s) => { },
  Error::Other => { },
  // 加上 `_ => {},` 就能编译通过
}

// 不包含通配符匹配臂，无法匹配非穷尽结构体
if let Ok(Config { window_width, window_height }) = config {
    // 加上 `..` 就能编译通过
}

match message {
  // 没有通配符，无法匹配非穷尽（结构体/枚举内的）变体
  Message::Send { from, to, contents } => { },
  // 无法匹配非穷尽元组或单元枚举变体(unit enum variant)。
  Message::Reaction(type) => { },
  Message::Quit => { },
}
```

非穷尽类型最好放在下游 crate 里。

[_EnumerationVariantExpression_]: ../expressions/enum-variant-expr.md
[_MetaWord_]: ../attributes.md#meta-item-attribute-syntax
[_StructExpression_]: ../expressions/struct-expr.md
[_StructPattern_]: ../patterns.md#struct-patterns
[_TupleStructPattern_]: ../patterns.md#tuple-struct-patterns
[`if let`]: ../expressions/if-expr.md#if-let-expressions
[属性]: ../attributes.md
[枚举变体]: ../items/enumerations.md
[函数更新句法]: ../expressions/struct-expr.md#functional-update-syntax
[结构体]: ../items/structs.md
