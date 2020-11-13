# Closure expressions
# 闭包表达式

>[closure-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/closure-expr.md)\
>commit: b2d11240bd9a3a6dd34419d0b0ba74617b23d77e \
>本章译文最后维护日期：2020-11-13

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

*闭包表达式*，也被称为 lambda表达式或 lambda。每个闭包表达式都定义了一个单独类型的闭包，其值就是闭包本身。闭包表达式是由管道定界符(`|`)封闭的不可反驳型[模式][patterns]参数列表，再后跟一个表达式组成。可以选择为参数类型或返回类型添加类型标注(type annotations)。如果存在返回类型，则用于闭包主体的表达式必须是一个普通的[块][block]。闭包表达式也可以以位于起始的 `|` 之前的关键字 `move` 为开头。

闭包表达式表示了将一组参数映射到参数后面的表达式的函数。与 [`let`绑定][`let` binding]一样，这里的参数也是不可反驳型[模式][patterns]的，其类型标注是可选的，如果没有给出，则从上下文推断。每个闭包表达式都有一个唯一的匿名类型。

闭包表达式在将函数作为参数传递给其他函数时非常有用，它常被用作为定义和捕获独立函数(separate function)的便捷手段。

特别值得注意的是闭包表达式能*捕获它们被定义时的环境中的变量(capture their environment)*，而正常[函数定义][function definitions]则不能。如果没有关键字 `move`，闭包表达式将[推断它该如何从其环境中捕获每个变量][infers how it captures each variable from its environment]，它倾向于通过共享引用来捕获，从而有效地借用闭包体中用到的所有外部变量。如果有必要，编译器将推断出应该采用可变引用，或者应该从环境中移动或复制值（取决于这些变量的类型）。闭包可以通过前缀关键字 `move` 来强制通过复制值或移动值的方式捕获其环境变量。这通常是为了确保当前闭包的生存期类型为 `'static`。

编译器将通过闭包对其捕获的变量的处置方式来确定此闭包类型将实现的[闭包trait][closure traits]。如果所有捕获的类型都实现了 [`Send`] 和/或 [`Sync`]，那么此闭包类型也实现了 `Send` 和/或 `Sync`。这些存在这些 trait，函数可以通过泛型的方式接受各种闭包，即便闭包的类型名无法被确切指定。

在本例中，我们定义了一个名为 `ten_times` 的函数，它接受高阶函数参数，然后我们传给它一个闭包表达式作为实参并调用它。之后又定义了一个使用移动语义从环境中捕获变量的闭包表达式来供该函数调用。

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

[infers how it captures each variable from its environment]: ../types/closure.md#capture-modes
[closure traits]: ../types/closure.md#call-traits-and-coercions
[`Send`]: ../special-types-and-traits.md#send
[`Sync`]: ../special-types-and-traits.md#sync
<!-- 上面这几个链接从原文来替换时需小心 -->
[block]: block-expr.md
[function definitions]: ../items/functions.md
[patterns]: ../patterns.md
[regular function parameters]: ../items/functions.md#attributes-on-function-parameters

[_Expression_]: ../expressions.md
[_BlockExpression_]: block-expr.md
[_TypeNoBounds_]: ../types.md#type-expressions
[_Pattern_]: ../patterns.md
[_Type_]: ../types.md#type-expressions
[`let` binding]: ../statements.md#let-statements
[_OuterAttribute_]: ../attributes.md

<!-- 2020-11-12-->
<!-- checked -->
