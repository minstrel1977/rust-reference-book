# 路径表达式

>[path-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/path-expr.md)\
>commit: b0e0ad6490d6517c19546b1023948986578fc378

> **<sup>句法</sup>**\
> _PathExpression_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_PathInExpression_]\
> &nbsp;&nbsp; | [_QualifiedPathInExpression_]

被用在表达式上下文里的[路径]表示局部变量或数据项。解析为局部变量或静态变量的路径表达式是[位置表达式]，其他路径是[值表达式]。使用 [`static mut`] 变量时需要引入 [`unsafe`块]。

```rust
# mod globals {
#     pub static STATIC_VAR: i32 = 5;
#     pub static mut STATIC_MUT_VAR: i32 = 7;
# }
# let local_var = 3;
local_var;
globals::STATIC_VAR;
unsafe { globals::STATIC_MUT_VAR };
let some_constructor = Some::<i32>;
let push_integer = Vec::<i32>::push;
let slice_reverse = <[i32]>::reverse;
```

[_PathInExpression_]: ../paths.md#表达式中的路径
[_QualifiedPathInExpression_]: ../paths.md#qualified-paths
[位置表达式]: ../expressions.md#位置表达式和值表达式
[值表达式]: ../expressions.md#位置表达式和值表达式
[路径]: ../paths.md
[`static mut`]: ../items/static-items.md#可变静态项
[`unsafe`块]: block-expr.md#unsafe块
