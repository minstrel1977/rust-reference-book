# Grouped expressions
# 分组表达式

>[grouped-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/grouped-expr.md)\
>commit: b0e0ad6490d6517c19546b1023948986578fc378 \
>本章译文最后维护日期：2020-10-26


> **<sup>句法</sup>**\
> _GroupedExpression_ :\
> &nbsp;&nbsp; `(` [_InnerAttribute_]<sup>\*</sup> [_Expression_] `)`

由圆括号封闭的表达式的求值结果就是在期内的表达式的求值结果。在表达式内部，圆括号可用于显式地指定表达式内部的求值顺序。

圆括号表达式的一个例子：

```rust
let x: i32 = 2 + 3 * 4;
let y: i32 = (2 + 3) * 4;
assert_eq!(x, 14);
assert_eq!(y, 20);
```

当调用结构体的函数指针类型的成员时，必须使用括号，示例如下：

```rust
# struct A {
#    f: fn() -> &'static str
# }
# impl A {
#    fn f(&self) -> &'static str {
#        "The method f"
#    }
# }
# let a = A{f: || "The field f"};
#
assert_eq!( a.f (), "The method f");
assert_eq!((a.f)(), "The field f");
```

## Group expression attributes
## 分组表达式上的属性

在与[块表达式上的属性][Inner attributes]相同的表达式上下文中，允许在分组表达式的左括号后直接使用[内部属性][attributes on block expressions]。

[Inner attributes]: ../attributes.md
[_Expression_]: ../expressions.md
[_InnerAttribute_]: ../attributes.md
[attributes on block expressions]: block-expr.md#attributes-on-block-expressions

<!-- 2020-11-7-->
<!-- checked -->
