# Enumeration Variant expressions
# 枚举变体表达式

>[enum-variant-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/enum-variant-expr.md)\
>commit 1a3615102993e9f017a44b903ff2277a38a171a8

> **<sup>句法</sup>**\
> _EnumerationVariantExpression_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _EnumExprStruct_\
> &nbsp;&nbsp; | _EnumExprTuple_\
> &nbsp;&nbsp; | _EnumExprFieldless_
>
> _EnumExprStruct_ :\
> &nbsp;&nbsp; [_PathInExpression_] `{` _EnumExprFields_<sup>?</sup> `}`
>
> _EnumExprFields_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _EnumExprField_ (`,` _EnumExprField_)<sup>\*</sup> `,`<sup>?</sup>
>
> _EnumExprField_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [IDENTIFIER]\
> &nbsp;&nbsp; | ([IDENTIFIER] | [TUPLE_INDEX]) `:` [_Expression_]
>
> _EnumExprTuple_ :\
> &nbsp;&nbsp; [_PathInExpression_] `(`\
> &nbsp;&nbsp; &nbsp;&nbsp; ( [_Expression_] (`,` [_Expression_])<sup>\*</sup> `,`<sup>?</sup> )<sup>?</sup>\
> &nbsp;&nbsp; `)`
>
> _EnumExprFieldless_ : [_PathInExpression_]

枚举变体的构造与[结构体][structs]的构造类似，只是使用枚举变体的路径来替代结构体的路径：

```rust
# enum Message {
#     Quit,
#     WriteString(String),
#     Move { x: i32, y: i32 },
# }
let q = Message::Quit;
let w = Message::WriteString("Some string".to_string());
let m = Message::Move { x: 50, y: 200 };
```

枚举变体表达式具有与[结构体表达式][structs]相同的语法、行为和限制，除了它不支持使用 `..`句法。

[IDENTIFIER]: ../identifiers.md
[TUPLE_INDEX]: ../tokens.md#元组索引
[_Expression_]: ../expressions.md
[_PathInExpression_]: ../paths.md#表达式中的路径
[structs]: struct-expr.md
