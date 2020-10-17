# 字面量表达式

>[literal-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/literal-expr.md)\
>commit: b0e0ad6490d6517c19546b1023948986578fc378

> **<sup>句法</sup>**\
> _LiteralExpression_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [CHAR_LITERAL]\
> &nbsp;&nbsp; | [STRING_LITERAL]\
> &nbsp;&nbsp; | [RAW_STRING_LITERAL]\
> &nbsp;&nbsp; | [BYTE_LITERAL]\
> &nbsp;&nbsp; | [BYTE_STRING_LITERAL]\
> &nbsp;&nbsp; | [RAW_BYTE_STRING_LITERAL]\
> &nbsp;&nbsp; | [INTEGER_LITERAL]\
> &nbsp;&nbsp; | [FLOAT_LITERAL]\
> &nbsp;&nbsp; | [BOOLEAN_LITERAL]

*字面量表达式*由上面的句法规则里给定的[字面量](../tokens.md#字面量)形式之一组成。它直接描述数字、字符、字符串或布尔值。

```rust
"hello";   // 字符串类型
'5';       // 字符类型
5;         // 整型
```

[CHAR_LITERAL]: ../tokens.md#字符字面量
[STRING_LITERAL]: ../tokens.md#字符串字面量
[RAW_STRING_LITERAL]: ../tokens.md#原生字符串字面量
[BYTE_LITERAL]: ../tokens.md#字节字面量
[BYTE_STRING_LITERAL]: ../tokens.md#字节串字面量
[RAW_BYTE_STRING_LITERAL]: ../tokens.md#原生字节串字面量
[INTEGER_LITERAL]: ../tokens.md#整型字面量
[FLOAT_LITERAL]: ../tokens.md#浮点型字面量
[BOOLEAN_LITERAL]: ../tokens.md#布尔型字面量
