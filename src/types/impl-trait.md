# Impl trait

>[impl-object.md](https://github.com/rust-lang/reference/blob/master/src/types/impl-object.md)\
>commit: 94085bbf726d53f3c7ace1ce122c82ab7b691d39 \
>本章译文最后维护日期：2020-5-29

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

```rust,ignore
trait Trait {}

// 泛型类型参数
fn foo<T: Trait>(arg: T) {
}

// 参数位置上的trait
fn foo(arg: impl Trait) {
}
```

也就是说，参数位置上的 `impl Trait` 是泛型类型参数（如 `<T: Trait>`）的语法糖，只是该类型是匿名的，并且不出现在 [_GenericParams_] 列表中。

> **注意：**
>对于函数参数，泛型类型参数和 `impl Trait` 并不完全等价。
>使用诸如 `<T:Trait>` 之类的泛型参数，调用者可以在调用点使用 [_GenericArgs_]（例如，`foo:：<usize>（1）`）形式来显式指定 `T` 的泛型参数。
>如果 `impl Trait` 是*任意*函数参数的类型，则调用者在调用该函数时不能提供任何泛型参数。这同样适用于返回类型或任何常量泛型的泛型参数。
>
>因此，这样将函数签名从一个变更为另一个，对函数的调用者来说仍可能是一种破坏性的变更。

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

### Differences between generics and `impl Trait` in return position
### 泛型与 `impl Trait` 在返回位置上的差异

在参数位置，`impl Trait` 在语义上与泛型类型参数非常相似。
然而，两者在返回位置上存在显著差异。
与泛型类型参数不同，使用 `impl Trait` 时，函数选择返回类型，调用者不能选择返回类型。

泛型函数：
```rust,ignore
fn foo<T: Trait>() -> T {
```
允许调用者来指定返回类型`T`，然后函数返回该类型。

`impl Trait` 函数：
```rust,ignore
fn foo() -> impl Trait {
```
不允许调用者指定返回类型。
相反，函数自身选择返回类型，但只承诺返回类型将实现 `Trait`。

## Limitations
## 限制

`impl Trait` 只能作为自由函数或固有函数的参数或返回类型出现。
它不能出现在 trait实现中，也不能出现在 let绑定的类型或出现在类型别名中。

[closures]: closure.md
[_GenericArgs_]: ../paths.md#paths-in-expressions
[_GenericParams_]: ../items/generics.md
[_TraitBound_]: ../trait-bounds.md
[trait object]: trait-object.md
[_TypeParamBounds_]: ../trait-bounds.md
