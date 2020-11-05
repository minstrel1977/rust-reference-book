# Field access expressions
# 字段访问表达式

>[field-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/field-expr.md)\
>commit: f8e76ee9368f498f7f044c719de68c7d95da9972 \
>本译文最后维护日期：2020-10-27

> **<sup>句法</sup>**\
> _FieldExpression_ :\
> &nbsp;&nbsp; [_Expression_] `.` [IDENTIFIER]

*字段表达式(field expression)*由一个表达式、一个单点号(`.`)和一个[标识符][identifier]组成，且后面不能再紧跟着一个被圆括号封闭起来的表达式列表（后者总是一个[方法调用表达式][method call expression]）。字段表达式代表[结构体(`struct`)][struct]或[联合体(`union`)][union]的字段。要调用存储在结构体的字段中的函数，需要在字段表达式外加一对圆括号。

<!-- ignore: needs lots of support code -->
```rust,ignore
mystruct.myfield;
foo().x;
(Struct {a: 10, b: 20}).a;
mystruct.method();          // 方法表达式
(mystruct.function_field)() // 调用表达式里包含一个字段表达式
```

字段访问是引用该字段位置的[位置表达式][place expression]。当子表达式是[可变的][mutable]时，此字段表达式也是可变的。

另外，如果点号左侧的表达式类型是指针，则会根据需要自动应用多次解引用来使字段访问成为可能。在存在歧义的情况下，Rust 倾向于较少次数的自动解引用。

最后，当用于借用时，对结构体的各个字段的借用或对整个结构体的引用都被视为彼此分离的实体。如果结构体没有实现 [`Drop`][`Drop`]，同时该结构体存储在局部变量中，（这种各个字段被视为彼此分离的单独实体的逻辑）还让每个字段的移出（move out）互不影响。如果对该结构体实现了户定义的自动解引用，这（种各个字段被视为彼此分离的单独实体的逻辑）就也不适用了。

```rust
struct A { f1: String, f2: String, f3: String }
let mut x: A;
# x = A {
#     f1: "f1".to_string(),
#     f2: "f2".to_string(),
#     f3: "f3".to_string()
# };
let a: &mut String = &mut x.f1; // x.f1 被可变借用
let b: &String = &x.f2;         // x.f2 被不可变借用
let c: &String = &x.f2;         // 可以被再次借用
let d: String = x.f3;           // 从 x.f3 中移出
```

[`Drop`]: ../special-types-and-traits.md#drop
<!-- 上面这几个链接从原文来替换时小心 -->
[_Expression_]: ../expressions.md
[IDENTIFIER]: ../identifiers.md
[method call expression]: method-call-expr.md
[struct]: ../items/structs.md
[union]: ../items/unions.md
[place expression]: ../expressions.md#place-expressions-and-value-expressions
[mutable]: ../expressions.md#mutability

<!-- 2020-11-3 -->
<!-- checked -->
