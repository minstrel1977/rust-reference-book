# Grouped expressions
# 分组表达式

>[grouped-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/grouped-expr.md)\
>commit b0e0ad6490d6517c19546b1023948986578fc378


> **<sup>句法</sup>**\
> _GroupedExpression_ :\
> &nbsp;&nbsp; `(` [_InnerAttribute_]<sup>\*</sup> [_Expression_] `)`

括在圆括号中的表达式的计算结果是封闭表达式的结果。在表达式内部，圆括号可用于显式地指定子表达式的求值顺序。

括号表达式的一个例子：

```rust
let x: i32 = 2 + 3 * 4;
let y: i32 = (2 + 3) * 4;
assert_eq!(x, 14);
assert_eq!(y, 20);
```

当调用作为结构体成员的函数指针时，必须使用括号，示例如下：

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

在与[块表达式上的属性]相同的表达式上下文中，允许在分组表达式的左括号后直接使用[内部属性]。
[Inner attributes] are allowed directly after the opening parenthesis of a
group expression in the same expression contexts as [attributes on block
expressions].

[内部属性]: ../attributes.md
[_Expression_]: ../expressions.md
[_InnerAttribute_]: ../attributes.md
[块表达式上的属性]: block-expr.md#块表达式上的属性
