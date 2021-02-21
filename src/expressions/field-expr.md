# Field access expressions
# 字段访问表达式

>[field-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/field-expr.md)\
>commit: eb5290329316e96c48c032075f7dbfa56990702b \
>本章译文最后维护日期：2021-02-21

> **<sup>句法</sup>**\
> _FieldExpression_ :\
> &nbsp;&nbsp; [_Expression_] `.` [IDENTIFIER]

*字段表达式(field expression)*由一个表达式，后跟一个单点号(`.`)和一个[标识符][identifier]组成，此时后面不能再紧跟着一个被圆括号封闭起来的表达式列表（后者是一个[方法调用表达式][method call expression]）。
字段表达式代表[结构体(`struct`)][struct]或[联合体(`union`)][union]的字段。要调用存储在结构体的字段中的函数，需要在此字段表达式外加上圆括号。

<!-- ignore: needs lots of support code -->
```rust,ignore
mystruct.myfield;
foo().x;
(Struct {a: 10, b: 20}).a;
mystruct.method();          // 方法表达式
(mystruct.function_field)() // 调用表达式里包含一个字段表达式
```

字段访问是用一个[位置表达式][place expression]去引用该字段的内存位置。
当字段表达式的子表达式是[可变的][mutable]时，此字段表达式也是可变的。

另外，如果点号左侧的表达式类型是指针型的，则会根据需要自动对其多次应用解引用来使字段访问成为可能。
在存在二义性的情况下，Rust 倾向于较少次数的自动解引用。

最后，当借用时，结构体的各个字段以及对结构体的整体引用都被视为彼此分离的实体。
如果结构体没有实现 [`Drop`]，同时该结构体又存储在局部变量中，（这种各个字段被视为彼此分离的单独实体的逻辑）还适用于每个字段的移出(move out)。
如果对 [`Box`]类型之外的用户定义的类型执行自动解引用，这（种各个字段被视为彼此分离的单独实体的逻辑）就不适用了。

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
[`Box`]: ../special-types-and-traits.md#boxt
<!-- 上面这几个链接从原文来替换时需小心 -->
[_Expression_]: ../expressions.md
[IDENTIFIER]: ../identifiers.md
[method call expression]: method-call-expr.md
[struct]: ../items/structs.md
[union]: ../items/unions.md
[place expression]: ../expressions.md#place-expressions-and-value-expressions
[mutable]: ../expressions.md#mutability