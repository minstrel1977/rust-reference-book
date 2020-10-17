# Closure expressions
# 闭包表达式

>[closure-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/closure-expr.md)\
>commit: b2d11240bd9a3a6dd34419d0b0ba74617b23d77e

> **<sup>句法</sup>**\
> _ClosureExpression_ :\
> &nbsp;&nbsp; `move`<sup>?</sup>\
> &nbsp;&nbsp; ( `||` | `|` _ClosureParameters_<sup>?</sup> `|` )\
> &nbsp;&nbsp; ([_Expression_] | `->` [_TypeNoBounds_]&nbsp;[_BlockExpression_])
>
> _ClosureParameters_ :\
> &nbsp;&nbsp; _ClosureParam_ (`,` _ClosureParam_)<sup>\*</sup> `,`<sup>?</sup>
>
> _ClosureParam_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> [_Pattern_]&nbsp;( `:` [_Type_] )<sup>?</sup>

*闭包表达式*，也称为 lambda表达式或 lambda，在单个表达式中定义闭包并将其表示为值。闭包表达式是由管道符号(`|`)封闭的不可反驳型[模式][patterns]的列表，后跟一个表达式。可以选择为参数类型或返回类型添加类型标注。如果存在返回类型，则用于闭包主体的表达式必须是普通[块][block]。闭包表达式也可以在初始 `|` 之前以关键字 `move` 开头。

闭包表达式表示将一组参数映射到参数后面的表达式的函数。与[`let`绑定][`let` binding]一样，这里的参数也是不可反驳型[模式][patterns]，其类型标注是可选的，如果没有给出，则从上下文推断。每个闭包表达式都有一个唯一的匿名类型。

闭包表达式在将函数作为参数传递给其他函数时最有用，它是定义和捕获独立函数的简称。

值得注意的是，闭包表达式能*捕获它们的环境*，而正常函数定义则不能。如果没有 `move` 关键字，闭包表达式将[推断出它该如何从其环境中捕获每个变量](../types/closure.md#capture-modes)，它会而倾向于通过共享引用来捕获，从而有效地借用闭包体中提到的所有外部变量。如果需要，编译器将推断应该采用可变引用，或者应该从环境中移动或复制值(取决于它们的类型)变量。闭包可以通过前缀 `move` 关键字来强制通过复制值或移动值的方式捕获其环境变量。这通常用来确保当前闭包的类型为 `'static`。

编译器将通过闭包对其捕获的变量的处置方式来确定闭包类型将实现哪些[闭包trait](../types/closure.md#call-traits-and-coercions)。如果所有捕获的类型都实现了 [`Send`](../special-types-and-traits.md#send) 和/或 [`Sync`](../special-types-and-traits.md#sync)，那么闭包也会实现 `Send` 和/或 `Sync`。即使不能指定确切的类型,这些 trait 也允许函数使用泛型接受闭包。

在本例中，我们定义了一个名为 `ten_times` 的函数，它接受高阶函数参数。然后用一个闭包表达式作为参数来调用它。最后还搞了一个从其环境中移进值的闭包表达式来供该函数调用。

```rust
fn ten_times<F>(f: F) where F: Fn(i32) {
    for index in 0..10 {
        f(index);
    }
}

ten_times(|j| println!("hello, {}", j));
// 带类型标注 i32
ten_times(|j: i32| -> () { println!("hello, {}", j) });

let word = "konnichiwa".to_owned();
ten_times(move |j| println!("{}, {}", word, j));
```

## Attributes on closure parameters
## 闭包参数上的属性

闭包参数上的属性遵循与[常规函数参数][regular function parameters]上相同的规则和限制。

[block]: block-expr.md
[function definitions]: ../items/functions.md
[patterns]: ../patterns.md
[regular function parameters]: ../items/functions.md#函数参数上的属性
[_Expression_]: ../expressions.md
[_BlockExpression_]: block-expr.md
[_TypeNoBounds_]: ../types.md#type-expressions
[_Pattern_]: ../patterns.md
[_Type_]: ../types.md#type-expressions
[`let` binding]: ../statements.md#let语句
[_OuterAttribute_]: ../attributes.md
<!-- 2020-10-16 -->