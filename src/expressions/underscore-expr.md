# `_` expressions
# `_`表达式

>[underscore-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/operator-expr.md)\
>commit: 78cd7345599c5ae65f21ac7d3127e7b0d1f6e707 \
>本章译文最后维护日期：2024-08-17

> **<sup>句法</sup>**\
> _UnderscoreExpression_ :\
> &nbsp;&nbsp; `_`

下划线表达式（使用符号 `_` 来表示）通常在解构赋值中的充当占位符。
它们只能出现在赋值表达式的左侧。

请注意，这与[通配符模式](../patterns.md#wildcard-pattern)是不同的。

`_`表达式的示例：

```rust
let p = (1, 2);
let mut a = 0;
(_, a) = p;

struct Position {
    x: u32,
    y: u32,
}

Position { x: a, y: _ } = Position{ x: 2, y: 3 };

// 把未使用的结果值赋给 `_` 一般用于声明意图和移除告警used to declare intent and remove a warning
_ = 2 + 2;
// 触发 unused_must_use 告警
// 2 + 2;

// 等效于在let绑定中使用通配符模式。
let _ = 2 + 2;
```
