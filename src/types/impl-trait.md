# Impl trait

>[impl-object.md](https://github.com/rust-lang/reference/blob/master/src/types/impl-object.md)\
>commit: fcacb13cf9eccce3596e11a841bd7d8528a2921c \
>本章译文最后维护日期：2024-10-13

> **<sup>句法</sup>**\
> _ImplTraitType_ : `impl` [_TypeParamBounds_]
>
> _ImplTraitTypeOneBound_ : `impl` [_TraitBound_]

`impl Trait` 提供了指定实现某trait 的具体的但匿名的类型的方法。
它可以出现在两种位置上：参数位置（在这里它可以充当函数的匿名类型参数）和返回位置（在这里它可以充当抽象返回类型）。

```rust
trait Trait {}
# impl Trait for () {}

// 参数位置：匿名类型参数
fn foo(arg: impl Trait) {
}

// 返回位置：抽象返回类型
fn bar() -> impl Trait {
}
```
## Anonymous type parameters
## 匿名类型参数

> 注意：匿名类型参数通常被称为“参数位置上的 trait约束(impl Trait in argument position)”。
(术语“参数”在这里更正式和正确，但 “impl Trait in argument position” 是在开发该特性时就使用的措辞，所以它仍保留在部分实现中。)

函数可以使用 `impl` 后跟一组 trait约束，将参数声明为具有某匿名类型。
调用者必须提供一个满足匿名类型参数声明的约束的类型，并且在函数内只能使用该匿名参数类型的 trait约束内部声明的方法。

例如，下面两种形式几乎等价：

```rust
trait Trait {}

// 泛型类型参数
fn with_generic_type<T: Trait>(arg: T) {
}

// 参数位置上的trait
fn with_impl_trait(arg: impl Trait) {
}
```

也就是说，参数位置上的 `impl Trait` 是泛型类型参数（如 `<T: Trait>`）的语法糖，只是该类型是匿名的，并且不出现在 [_GenericParams_] 列表中。

> **注意：**
>对于函数参数，泛型类型参数和 `impl Trait` 并不完全等价。
>使用诸如 `<T:Trait>` 之类的泛型参数，调用者可以在调用点使用 [_GenericArgs_]（例如，`foo:：<usize>（1）`）形式来显式指定 `T` 的泛型参数。
> 将参数从一个更改为另一个可能会构成函数调用的破坏性改变，因为这会更改泛型参数的数量。

## Abstract return types
## 抽象返回类型

> 注意： 这通常被称为“返回位置上的 Trait约束”(impl Trait in return position)。

函数可以使用 `impl Trait` 返回抽象返回类型。
这些类型代表另一个具体类型，调用者只能使用指定的 `Trait` 内声明的方法。
函数的每个可能的返回分支返回的返回值都必须解析为相同的具体类型。

返回位置上的 `impl Trait` 允许函数返回不用装箱的抽象类型。
这对于[闭包][closures]和迭代器特别有用。
例如，闭包具有唯一的、不可写的类型。
以前，从函数返回闭包的唯一方法是使用[trait对象][trait object]：

```rust
fn returns_closure() -> Box<dyn Fn(i32) -> i32> {
    Box::new(|x| x + 1)
}
```

这不得不承受因堆分配和动态调度而带来的性能损失。
并且者无法完全指定闭包的类型，只能使用 `Fn` trait。
这意味着此时 trait对象是必要的。
但是，使用 `impl Trait`，可以像如下这样来更简单地编写代码：

```rust
fn returns_closure() -> impl Fn(i32) -> i32 {
    |x| x + 1
}
```

这也避免了对 trait对象进行装箱的缺陷。

类似地，在迭代器的使用坏境中，如果将此步操作前的所有迭代器操作合并到一个执行链中，迭代器里的具体类型可能变得非常复杂。此时返回 `impl Iterator` 意味着一个函数只需将 `Iterator` trait 作为其返回类型的约束来公开，而不须显式指定链内所有其他涉及的迭代器类型。

## Return-position `impl Trait` in traits and trait implementations
## trait 和 trait实现 的返回位置上的`impl Trait`

trait中的函数也可以使用 `impl Trait` 作为匿名关联类型的句法。

trait中关联函数的返回类型中的每个 `impl Trait` 都被脱糖为匿名关联类型。出现在实现中的函数签名中的返回类型用于确定关联类型的确定类型。

## Capturing
## 捕获

在每个返回位置上的 `impl Trait` 抽象类型后面是一些隐藏的具体类型。为了使该具体类型使用泛型参数，该泛型参数必须由抽象类型*捕获*。

## Automatic capturing
## 自动捕获

返回位置上的 `impl Trait` 抽象类型会自动捕获在当前作用域内生效的泛型参数。在任何地方，这些都会自动捕获在当前作用域内生效的类型参数和常量泛型参数。

在 trait实现上和 trait定义的程序项上，这些类型另外能自动捕获在当前作用域内生效的范型生存期参数，包括高阶的参数。在自由函数、关联函数和固有实现的方法上，仅能捕获出现在抽象返回类型约束中的泛型生存期参数。

## Precise capturing
## 精确捕获

由返回位置上的 `impl Trait`抽象类型捕获的泛型参数集可以用 [`use<..>` bound] 显式控制。如果存在 `use<..>`，则仅在 `use<..>`约束 中列出的泛型参数会被捕获。例如：

```rust
fn capture<'a, 'b, T>(x: &'a (), y: T) -> impl Sized + use<'a, T> {
  //                                      ~~~~~~~~~~~~~~~~~~~~~~~
  //                                     Captures `'a` and `T` only.
  (x, y)
}
```

当前，约束列表中只能出现一个 `use<..>`约束，有 `use<..>`的约束不允许出现在 trait定义的各种程序项的签名中，定义trait时，必须包括所有进入作用域内的类型参数和常量泛型参数，并且必须包括出现在抽象类型的其他约束中的所有生存期参数。在`use<..>`内部，出现的的任何生存期参数都必须放在所有类型参数和常量泛型参数之前，并且如果允许自动推导的生存期(`'_`)出现在 `impl Trait`返回类型中，则它也要遵守这个规则。

由于所有能进入当前作用域内的类型参数都必须通过名称进入的，因此 `use<..>`约束不能用于那些带有参数位置上的 `impl Trait`的程序项的签名，因为这些项在作用域中具有匿名类型参数。

## Differences between generics and `impl Trait` in return position
## 泛型与 `impl Trait` 在返回位置上的差异

在参数位置，`impl Trait` 在语义上与泛型类型参数非常相似。
然而，两者在返回位置上存在显著差异。
与泛型类型参数不同，使用 `impl Trait` 时，函数选择返回类型，调用者不能选择返回类型。

泛型函数：
```rust
# trait Trait {}
fn foo<T: Trait>() -> T {
    // ...
# panic!()
}
```
允许调用者来指定返回类型`T`，然后函数返回该类型。

`impl Trait` 函数：
```rust
# trait Trait {}
# impl Trait for () {}
fn foo() -> impl Trait {
    // ...
}
```
不允许调用者指定返回类型。
相反，函数自身选择返回类型，但只承诺返回类型将实现 `Trait`。

## Limitations
## 限制

`impl Trait` 只能作为 非`extern`函数的参数类型或返回类型出现。
它不能 `let`绑定、成员字段的类型，也不能出现在类型别名中。

[_GenericArgs_]: ../paths.md#paths-in-expressions
[_GenericParams_]: ../items/generics.md
[_TraitBound_]: ../trait-bounds.md
[_TypeParamBounds_]: ../trait-bounds.md
[`use<..>` bound]: ../trait-bounds.md#use-bounds
[closures]: closure.md
[trait object]: trait-object.md
