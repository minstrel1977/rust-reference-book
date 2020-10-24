# Path expressions
# 路径表达式

>[path-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/path-expr.md)\
>commit: b0e0ad6490d6517c19546b1023948986578fc378 \
>本译文最后维护日期：2020-10-24

> **<sup>句法</sup>**\
> _PathExpression_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_PathInExpression_]\
> &nbsp;&nbsp; | [_QualifiedPathInExpression_]

被用做表达式上下文的[路径][path]表示局部变量或数据项。解析为局部变量或静态变量的路径表达式是[位置表达式][place expressions]，其他路径是[值表达式][value expressions]。使用 [`static mut`] 变量需要引入 [`unsafe`块][`unsafe` block]。

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

[_PathInExpression_]: ../paths.md#paths-in-expressions
[_QualifiedPathInExpression_]: ../paths.md#qualified-paths
[place expressions]: ../expressions.md#place-expressions-and-value-expressions
[value expressions]: ../expressions.md#place-expressions-and-value-expressions
[path]: ../paths.md
[`static mut`]: ../items/static-items.md#mutable-statics
[`unsafe` block]: block-expr.md#unsafe-blocks

<!-- 2020-10-16 -->
<!-- checked -->
