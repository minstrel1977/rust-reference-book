# Method-call expressions
# 方法调用表达式

>[method-call-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/method-call-expr.md)\
>commit: b0e0ad6490d6517c19546b1023948986578fc378 \
>本章译文最后维护日期：2020-10-27

> **<sup>句法</sup>**\
> _MethodCallExpression_ :\
> &nbsp;&nbsp; [_Expression_] `.` [_PathExprSegment_] `(`[_CallParams_]<sup>?</sup> `)`

*方法调用*由一个表达式（*接受者(receiver)*）后跟一个单点号(`.`)、一个表达式路径段(path segment)和一个圆括号封闭的的表达式列表组成。方法调用被解析为特定 trait 上的关联[方法][methods]时，如果点号左边表达式有确切的已知的 `self` 类型，则会静态地分发(dispatch)给某个查找到的同名方法来执行；如果点号左边表达式是间接的 [trait对象](https://doc.rust-lang.org/types/trait-object.md)，则会采用动态分发。

```rust
let pi: Result<f32, _> = "3.14".parse();
let log_pi = pi.unwrap_or(1.0).log(2.72);
# assert!(1.14 < log_pi && log_pi < 1.15)
```

在查找方法调用时，为了调用某个方法，可能会自动对接受者做解引用或借用。这需要比普通函数更复杂的查找流程，因为可能需要调用许多可能的方法。具体会用到下述步骤：

第一步是构建候选接受者类型的列表。通过重复对接收方表达式的类型作[解引用][dereference]，将遇到的每个类型添加到列表中，然后在最后再尝试进行一次[非固定尺寸类型自动强转(unsized coercion)][unsized coercion]，如果成功，则将结果类型也添加到类型列表。然后，再在这个列表中的每个候选类型 `T` 后紧跟着添加 `&T` 和 `&mut T` 候选项。

例如，接受者的类型为 `Box<[i32;2]>`，则候选类型为 `Box<[i32;2]>`，`&Box<[i32;2]>`，`&mut Box<[i32;2]>`，`[i32; 2]`（通过解引用得到），`&[i32; 2]`，`&mut [i32; 2]`，`[i32]`（通过非固定尺寸类型自动强转得到），`&[i32]`，最后是 `&mut [i32]`。

然后，对每个候选类型 `T`，编译器会在以下位置搜索接受者为该候选类型的一个[可见的][visible]方法：

1. `T` 的固有方法（直接在 `T` 上实现的方法）。
2. 由 `T` 实现的[可见的][visible] trait 所提供的任何方法。如果 `T` 是一个类型参数，则首先查找由 `T` 上的 trait约束所提供的方法。然后查找作用域内所有其他的方法。

> 注意：查找是按顺序进行的，这有时会导致不太符合直觉的结果。下面的代码将打印 “In trait impl!”，因为首先会查找 `&self` 上的方法，在找到结构体 `Foo` 的（接受者为） `&mut self` 方法之前先找到（接受者为 `&self` 的 `Bar::bar`）trait方法。
>
> ```rust
> struct Foo {}
>
> trait Bar {
>   fn bar(&self);
> }
>
> impl Foo {
>   fn bar(&mut self) {
>     println!("In struct impl!")
>   }
> }
>
> impl Bar for Foo {
>   fn bar(&self) {
>     println!("In trait impl!")
>   }
> }
>
> fn main() {
>   let mut f = Foo{};
>   f.bar();
> }
> ```

如果查找导致了多个可能的候选项（译者注：比如接受者实现的两个 trait中都定义了该方法），就会导致报错，此时必须将接受者[转换][disambiguate call]为适当的接受者类型来进行方法调用。

此过程不考虑接受者的可变性或生存期，也不考虑方法是否为非安全(`unsafe`)方法。一旦查找到了一个方法，如果由于其中一个（或多个）原因而不能调用它，则会报编译错误。

如果碰到了某步存在多个可能性的方法，比如泛型方法之间或 trait方法之间被认为是相同的，那么它就会导致编译错误。这些情况就需要使用[消除函数调用歧义的句法][disambiguating function call syntax]为方法或函数的调用来消除歧义。

<div class="warning">

***警告：*** 对于 [trait对象][trait objects]，如果有一个与 trait方法同名的固有方法，那么当尝试在方法调用表达式中调用该方法时，将编译报错。此时，可以使用[消除函数调用歧义的句法][disambiguating function call syntax]来明确调用语义。在 trait对象上使用消除函数调用歧义的句法，将只能调用 trait方法，无法调用固有方法。所以只要不在 trait对象上定义和 trait方法同名的固有方法就不会碰到这种麻烦。

</div>

[trait object]: ../types/trait-object.md
<!-- 上面这几个链接从原文来替换时需小心 -->
[_CallParams_]: call-expr.md
[_Expression_]: ../expressions.md
[_PathExprSegment_]: ../paths.md#paths-in-expressions
[visible]: ../visibility-and-privacy.md
[trait objects]: ../types/trait-object.md
[disambiguate call]: call-expr.md#disambiguating-function-calls
[disambiguating function call syntax]: call-expr.md#disambiguating-function-calls
[dereference]: operator-expr.md#the-dereference-operator
[methods]: ../items/associated-items.md#methods
[unsized coercion]: ../type-coercions.md#unsized-coercions

<!-- 2020-11-7-->
<!-- checked -->
