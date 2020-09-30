# `return` expressions
# `return`表达式

>[return-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/return-expr.md)\
>commit b0e0ad6490d6517c19546b1023948986578fc378

> **<sup>句法</sup>**\
> _ReturnExpression_ :\
> &nbsp;&nbsp; `return` [_Expression_]<sup>?</sup>

返回表达式用关键字 `return` 表示。对 `return`表达式求值会将其参数移动到当前函数调用的指定输出位置，然后销毁当前的函数激活帧，并将控制权转移到调用者所在的帧。

一个 `return`表达式的例子：

```rust
fn max(a: i32, b: i32) -> i32 {
    if a > b {
        return a;
    }
    return b;
}
```

[_Expression_]: ../expressions.md
