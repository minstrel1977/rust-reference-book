# Tuple and tuple indexing expressions
# 元组和元组索引表达式

>[tuple-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/tuple-expr.md)\
>commit: 1a3615102993e9f017a44b903ff2277a38a171a8 \
>本章译文最后维护日期：2020-10-26

## Tuple expressions
## 元组表达式

> **<sup>句法</sup>**\
> _TupleExpression_ :\
> &nbsp;&nbsp; `(` [_InnerAttribute_]<sup>\*</sup> _TupleElements_<sup>?</sup> `)`
>
> _TupleElements_ :\
> &nbsp;&nbsp; ( [_Expression_] `,` )<sup>+</sup> [_Expression_]<sup>?</sup>

元组是通过将零个或多个以逗号分隔的表达式括在圆括号中来编写的。可用它们来创建[元组类型][tuple-typed])的值。

```rust
(0.0, 4.5);
("a", 4usize, true);
();
```

可以用逗号消除单元素元组与括号表达式之间的歧义：

```rust
(0,); // 单个元素的元组
(0); // 0在括号中
```

### Tuple expression attributes
### 元组表达式上的属性

允许[块表达式上的属性][attributes on block expressions]的表达式上下文同样允许元组表达式上的属性，即在这种上下文下[内部属性][Inner attributes]可以直接位于表达式的左括号之后。

## Tuple indexing expressions
## 元组索引表达式

> **<sup>句法</sup>**\
> _TupleIndexingExpression_ :\
> &nbsp;&nbsp; [_Expression_] `.` [TUPLE_INDEX]

[元组][Tuples]和[元组结构体][struct tuples]可以使用与字段位置相对应的数字来编制索引。索引必须写成不带下划线或后缀的[十进制字面量][decimal literal]。元组索引表达式也不同于字段表达式，因为元组索引表达式可以明确地作为函数来调用。在所有其他方面，它们有相同的行为。

```rust
# struct Point(f32, f32);
let pair = (1, 2);
assert_eq!(pair.1, 2);
let unit_x = Point(1.0, 0.0);
assert_eq!(unit_x.0, 1.0);
```

[tuple-typed]: ../types/tuple.md
[Tuples]: ../types/tuple.md
[struct tuples]: ../items/structs.md
[decimal literal]: ../tokens.md#integer-literals
<!-- 上面这几个链接从原文来替换时需小心 -->
[Inner attributes]: ../attributes.md
[TUPLE_INDEX]: ../tokens.md#tuple-index
[_Expression_]: ../expressions.md
[_InnerAttribute_]: ../attributes.md
[attributes on block expressions]: block-expr.md#attributes-on-block-expressions

<!-- 2020-11-7-->
<!-- checked -->
