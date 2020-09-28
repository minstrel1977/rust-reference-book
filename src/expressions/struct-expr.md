# Struct expressions
# 结构体表达式

> **<sup>句法</sup>**\
> _StructExpression_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _StructExprStruct_\
> &nbsp;&nbsp; | _StructExprTuple_\
> &nbsp;&nbsp; | _StructExprUnit_
>
> _StructExprStruct_ :\
> &nbsp;&nbsp; [_PathInExpression_] `{` [_InnerAttribute_]<sup>\*</sup> (_StructExprFields_ | _StructBase_)<sup>?</sup> `}`
>
> _StructExprFields_ :\
> &nbsp;&nbsp; _StructExprField_ (`,` _StructExprField_)<sup>\*</sup> (`,` _StructBase_ | `,`<sup>?</sup>)
>
> _StructExprField_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [IDENTIFIER]\
> &nbsp;&nbsp; | ([IDENTIFIER] | [TUPLE_INDEX]) `:` [_Expression_]
>
> _StructBase_ :\
> &nbsp;&nbsp; `..` [_Expression_]
>
> _StructExprTuple_ :\
> &nbsp;&nbsp; [_PathInExpression_] `(`\
> &nbsp;&nbsp; &nbsp;&nbsp; [_InnerAttribute_]<sup>\*</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp; ( [_Expression_] (`,` [_Expression_])<sup>\*</sup> `,`<sup>?</sup> )<sup>?</sup>\
> &nbsp;&nbsp; `)`
>
> _StructExprUnit_ : [_PathInExpression_]

*结构体表达式*创建结构体或联合体的值。它由通向[结构体][struct]或[联合体][union]数据项的路径和数据项的字段的值组成。结构体表达式有三种形式：结构体、元组结构体和单元结构体。

下面是结构体表达式的示例：

```rust
# struct Point { x: f64, y: f64 }
# struct NothingInMe { }
# struct TuplePoint(f64, f64);
# mod game { pub struct User<'a> { pub name: &'a str, pub age: u32, pub score: usize } }
# struct Cookie; fn some_fn<T>(t: T) {}
Point {x: 10.0, y: 20.0};
NothingInMe {};
TuplePoint(10.0, 20.0);
TuplePoint { 0: 10.0, 1: 20.0 }; // 效果和上一行一样
let u = game::User {name: "Joe", age: 35, score: 100_000};
some_fn::<Cookie>(Cookie);
```

## Field struct expression
## 结构体表达式的字段设置


用花括号括起来的字段的结构体表达式允许您以任意顺序指定每个字段的值。字段名与值之间用冒号分隔。

[联合体][union]类型的值也可以使用此语法创建，但必须只能指定一个字段。

## Functional update syntax
## 函数式更新句法

结构体表达式可以以 `..` 后跟一个表达式的句法结尾，这种句法表示这是一种函数式更新。`..` 后跟的表达式（译者注：此表达式被称为此函数式更新的基，英文原文为：the base）必须是具有与正在构建的新结构体类型相同的结构体类型。

整个表达式为指定的字段使用给定的值，并从基表达式(base expression)移动或复制其余字段。与所有结构体表达式一样，结构体的所有字段必须是[可见的][visible]，甚至那些没有显式命名的字段也是如此。

```rust
# struct Point3d { x: i32, y: i32, z: i32 }
let mut base = Point3d {x: 1, y: 2, z: 3};
let y_ref = &mut base.y;
Point3d {y: 0, z: 10, .. base}; // OK, 只有 base.x 获取进来了
drop(y_ref);
```

带花括号的结构体表达式不能直接在[循环][loop]或 [if]表达式的头部，或在 [if let]或[匹配][match]表达式的[检验对象][scrutinee]中使用。但是，如果结构体表达式在另一个表达式内(例如在[圆括号][parentheses]内)，则可以在这些情况下使用。

字段名可以用十进制整数值来指定用于构造元组结构体的索引。这可以与基本结构体一起使用来填充其余未指定的索引：

```rust
struct Color(u8, u8, u8);
let c1 = Color(0, 0, 0);  // 创建元组结构体的典型方法。
let c2 = Color{0: 255, 1: 127, 2: 0};  // 按索引来指定字段。
let c3 = Color{1: 0, ..c2};  // 使用基的字段值来填写结构体的所有其他字段。
```

### Struct field init shorthand

When initializing a data structure (struct, enum, union) with named (but not
numbered) fields, it is allowed to write `fieldname` as a shorthand for
`fieldname: fieldname`. This allows a compact syntax with less duplication.
For example:

```rust
# struct Point3d { x: i32, y: i32, z: i32 }
# let x = 0;
# let y_value = 0;
# let z = 0;
Point3d { x: x, y: y_value, z: z };
Point3d { x, y: y_value, z };
```

## Tuple struct expression

A struct expression with fields enclosed in parentheses constructs a tuple struct. Though
it is listed here as a specific expression for completeness, it is equivalent to a [call
expression] to the tuple struct's constructor. For example:

```rust
struct Position(i32, i32, i32);
Position(0, 0, 0);  // Typical way of creating a tuple struct.
let c = Position;  // `c` is a function that takes 3 arguments.
let pos = c(8, 6, 7);  // Creates a `Position` value.
```

## Unit struct expression

A unit struct expression is just the path to a unit struct item. This refers to the unit
struct's implicit constant of its value. The unit struct value can also be constructed
with a fieldless struct expression. For example:

```rust
struct Gamma;
let a = Gamma;  // Gamma unit value.
let b = Gamma{};  // Exact same value as `a`.
```

## Struct expression attributes

[Inner attributes] are allowed directly after the opening brace or parenthesis
of a struct expression in the same expression contexts as [attributes on block
expressions].

[IDENTIFIER]: ../identifiers.md
[Inner attributes]: ../attributes.md
[TUPLE_INDEX]: ../tokens.md#元组索引
[_Expression_]: ../expressions.md
[_InnerAttribute_]: ../attributes.md
[_PathInExpression_]: ../paths.md#表达式中的路径
[attributes on block expressions]: block-expr.md#块表达式上的属性
[call expression]: call-expr.md
[if let]: if-expr.md#if-let-expressions
[if]: if-expr.md#if-expressions
[loop]: loop-expr.md
[match]: match-expr.md
[parentheses]: grouped-expr.md
[struct]: ../items/structs.md
[union]: ../items/unions.md
[visible]: ../visibility-and-privacy.md
[scrutinee]: ../glossary.md#scrutinee
