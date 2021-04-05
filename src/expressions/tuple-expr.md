# Tuple and tuple indexing expressions
# 元组和元组索引表达式

>[tuple-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/tuple-expr.md)\
>commit: eb5290329316e96c48c032075f7dbfa56990702b \
>本章译文最后维护日期：2021-02-21

## Tuple expressions
## 元组表达式

> **<sup>句法</sup>**\
> _TupleExpression_ :\
> &nbsp;&nbsp; `(` [_InnerAttribute_]<sup>\*</sup> _TupleElements_<sup>?</sup> `)`
>
> _TupleElements_ :\
> &nbsp;&nbsp; ( [_Expression_] `,` )<sup>+</sup> [_Expression_]<sup>?</sup>

<!-- 元组表达式是通过将零个或多个以逗号分隔的表达式括在圆括号中来编写的。可用它们来创建[元组类型][tuple type])的值。 -->
元组表达式的求值就是将其操作数初始化为[元组值][tuple type]的元素。

元组表达式的编写方法是在逗号分隔的圆括号列表中列出[操作数][operands]。
一元(1-ary)元组表达式的操作数后面需要一个逗号，以便能和[括号表达式][parenthetical expression]区分开来。

操作数的数量构成元组的元数(arity)。
没有操作数的元组表达式生成单元元组(unit tuple)。
对于其他元组表达式，第一个被写入的操作数初始化第 0 个元素，随后的操作数依次初始化下一个开始的元素。
例如，在元组表达式 `('a', 'b', 'c')` 中，`'a'` 初始化第 0 个元素的值，`'b'` 初始化第 1 个元素，`'c'` 初始化第2个元素。

元组表达式示例：

| 表达式                | 类型          |
| -------------------- | ------------ |
| `()`                 | `()` (unit)  |
| `(0.0, 4.5)`         | `(f64, f64)` |
| `("x".to_string(), )` | `(String, )`  |
| `("a", 4usize, true)`| `(&'static str, usize, bool)` |

### Tuple expression attributes
### 元组表达式上的属性

在允许[块表达式上的属性][Inner attributes]存在的那几种表达式上下文中，可以在元组表达式的左括号后直接使用[内部属性][attributes on block expressions]。

## Tuple indexing expressions
## 元组索引表达式

> **<sup>句法</sup>**\
> _TupleIndexingExpression_ :\
> &nbsp;&nbsp; [_Expression_] `.` [TUPLE_INDEX]

元组索引表达式的计算类似于[字段访问表达式][field access expressions]，但是访问的是[元组][tuple type]或[元组结构体][tuple structs]的元素。

元组索引表达式被写成操作数，后跟 `.`，以及一个元组索引。
索引必须写成[十进制字面量][decimal literal]的形式，不能有前导零、下划线和后缀。
元组索引表达式的操作数必须是元组或元组结构体类型。如果元组索引不是元组或元组结构体的元素，则报编译错误。

元组索引表达式示例：

```rust
let pair = ("a string", 2);
assert_eq!(pair.1, 2);

# struct Point(f32, f32);
let point = Point(1.0, 0.0);
assert_eq!(point.0, 1.0);
assert_eq!(point.1, 0.0);
```

> **注意**: 与字段访问表达式不同，元组索引表达式可以是[调用表达式][call expression]的函数操作数。
> （这之所以可行，）因为元组索引表达式不会与方法调用相混淆，因为方法名不可能是数字。

[_Expression_]: ../expressions.md
[_InnerAttribute_]: ../attributes.md
[attributes on block expressions]: block-expr.md#attributes-on-block-expressions
[call expression]: ./call-expr.md
[decimal literal]: ../tokens.md#integer-literals
[field access expressions]: ./field-expr.html#field-access-expressions
[Inner attributes]: ../attributes.md
[operands]: ../expressions.md
[parenthetical expression]: grouped-expr.md
[tuple type]: ../types/tuple.md
[tuple structs]: ../types/struct.md
[TUPLE_INDEX]: ../tokens.md#tuple-index
