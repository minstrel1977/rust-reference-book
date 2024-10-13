# Path expressions
# 路径表达式

>[path-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/path-expr.md)\
>commit: d33e4b03f0f810a315915412448a1f73c30e0feb \
>本章译文最后维护日期：2024-05-26

> **<sup>句法</sup>**\
> _PathExpression_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_PathInExpression_]\
> &nbsp;&nbsp; | [_QualifiedPathInExpression_]

[路径][path]被用做表达式上下文时表示局部变量或程序项。
解析为局部变量或静态变量的路径表达式是[位置表达式][place expressions]，其他路径是[值表达式][value expressions]。
使用 [`static mut`]变量需在 [`unsafe`块][`unsafe` block]中。

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

关联常量的计算处理方式与[`const`块][`const` blocks]相同。

[_PathInExpression_]: ../paths.md#paths-in-expressions
[_QualifiedPathInExpression_]: ../paths.md#qualified-paths
[place expressions]: ../expressions.md#place-expressions-and-value-expressions
[value expressions]: ../expressions.md#place-expressions-and-value-expressions
[path]: ../paths.md
[`static mut`]: ../items/static-items.md#mutable-statics
[`unsafe` block]: block-expr.md#unsafe-blocks
[`const` blocks]: block-expr.md#const-blocks
