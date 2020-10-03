# Tuple types
# 元组类型

>[tuple.md](https://github.com/rust-lang/reference/blob/master/src/types/tuple.md)\
>commit b0e0ad6490d6517c19546b1023948986578fc378

> **<sup>句法</sup>**\
> _TupleType_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `(` `)`\
> &nbsp;&nbsp; | `(` ( [_Type_] `,` )<sup>+</sup> [_Type_]<sup>?</sup> `)`

元组(tuple)*类型*是由其他类型异构组合的产物，这些其他类型被称为元组的*元素*。元组没有标称型(nominal)名称，取而代之的是直接键入其类型结构。

定义元组类型是通过直接在圆括号里的逗号分隔的列表中列出其元素的类型实现的。定义元组值可以不通过其类型，可以直接在圆括号里的逗号分隔的列表中列出其元素的值来实现。

因为元组元素没有名称，所以只能通过模式匹配或直接使用字段的序号 `N` 作为字段来访问第N个元素。

元组类型及其使用的示例：

```rust
type Pair<'a> = (i32, &'a str);
let p: Pair<'static> = (10, "ten");
let (a, b) = p;

assert_eq!(a, 10);
assert_eq!(b, "ten");
assert_eq!(p.0, 10);
assert_eq!(p.1, "ten");
```

由于历史原因和为了使用方便，代表没有元素(`()`)的元组类型通常被称为单元(`unit`)或单元类型(`the unit type`)。

[_Type_]: ../types.md#type-expressions
