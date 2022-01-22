# `_` expressions
# `_`表达式

>[underscore-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/operator-expr.md)\
>commit: 4115c9a3c2bd91f29445999891de4e8d26d60dc8 \
>本章译文最后维护日期：2022-01-22

> **<sup>句法</sup>**\
> _UnderscoreExpression_ :\
> &nbsp;&nbsp; `_`

下划线表达式（使用符号 `_` 来表示）通常在解构赋值中的充当占位符。
它们只能出现在赋值表达式的左侧。

一个 `_`表达式的一个示例：

```rust
let p = (1, 2);
let mut a = 0;
(_, a) = p;
```
