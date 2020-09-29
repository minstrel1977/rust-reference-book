# Field access expressions
# 字段存取表达式

>[field-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/field-call-expr.md)\
>commit f8e76ee9368f498f7f044c719de68c7d95da9972

> **<sup>句法</sup>**\
> _FieldExpression_ :\
> &nbsp;&nbsp; [_Expression_] `.` [IDENTIFIER]

*字段表达式*由一个表达式、一个单点号和一个[标识符][identifier]组成，且后面不能再紧跟着一个被圆括号括起来的表达式列表(后者总是一个[方法调用表达式][method call expression])。字段表达式代表[结构体][struct]或[联合体][union]的字段。要调用存储在结构体中的函数(型字段)，需要在字段表达式外加一对圆括号。

<!-- ignore: needs lots of support code -->
```rust,ignore
mystruct.myfield;
foo().x;
(Struct {a: 10, b: 20}).a;
mystruct.method();          // 方法表达式
(mystruct.function_field)() // 调用表达式里包含一个字段表达式
```

字段访问是引用该字段位置的[位置表达式][place expression]。当子表达式是[可变的][mutable]时，此字段表达式也是可变的。

另外，如果点号左侧的表达式类型是指针，则会根据需要自动多次解引用来使字段访问成为可能。在模棱两可的情况下，Rust 倾向于更少的自动解引用。

Finally, the fields of a struct or a reference to a struct are treated as
separate entities when borrowing. If the struct does not implement
[`Drop`](../special-types-and-traits.md#drop) and is stored in a local variable,
this also applies to moving out of each of its fields. This also does not apply
if automatic dereferencing is done though user defined types.

```rust
struct A { f1: String, f2: String, f3: String }
let mut x: A;
# x = A {
#     f1: "f1".to_string(),
#     f2: "f2".to_string(),
#     f3: "f3".to_string()
# };
let a: &mut String = &mut x.f1; // x.f1 borrowed mutably
let b: &String = &x.f2;         // x.f2 borrowed immutably
let c: &String = &x.f2;         // Can borrow again
let d: String = x.f3;           // Move out of x.f3
```

[_Expression_]: ../expressions.md
[IDENTIFIER]: ../identifiers.md
[method call expression]: method-call-expr.md
[struct]: ../items/structs.md
[union]: ../items/unions.md
[place expression]: ../expressions.md#位置表达式和值表达式
[mutable]: ../expressions.md#可变性
