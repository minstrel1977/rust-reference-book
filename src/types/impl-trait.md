# Impl trait

>[impl-object.md](https://github.com/rust-lang/reference/blob/master/src/types/impl-object.md)\
>commit: 94085bbf726d53f3c7ace1ce122c82ab7b691d39 \
>本章译文最后维护日期：2020-5-29

> **<sup>句法</sup>**\
> _ImplTraitType_ : `impl` [_TypeParamBounds_]
>
> _ImplTraitTypeOneBound_ : `impl` [_TraitBound_]

`impl Trait` 提供了指定实现某trait 的匿名但具体的类型的方法。
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
调用者必须提供一个满足匿名类型参数声明的约束的类型，并且在函数内只能使用该匿名参数类型的 trait约束里可用的方法。

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
>因此，这样将函数签名从一个更改为另一个，对函数的调用者来说仍可能是一种构成破坏性的更改。

## Abstract return types
## 抽象返回类型

> 注意： 这通常被称为“返回位置的Trait约束”(impl Trait in return position)。

Functions can use `impl Trait` to return an abstract return type.
These types stand in for another concrete type where the caller may only use the methods declared by the specified `Trait`.
Each possible return value from the function must resolve to the same concrete type.

`impl Trait` in return position allows a function to return an unboxed abstract type.
This is particularly useful with [closures] and iterators.
For example, closures have a unique, un-writable type.
Previously, the only way to return a closure from a function was to use a [trait object]:

```rust
fn returns_closure() -> Box<dyn Fn(i32) -> i32> {
    Box::new(|x| x + 1)
}
```

This could incur performance penalties from heap allocation and dynamic dispatch.
It wasn't possible to fully specify the type of the closure, only to use the `Fn` trait.
That means that the trait object is necessary.
However, with `impl Trait`, it is possible to write this more simply:

```rust
fn returns_closure() -> impl Fn(i32) -> i32 {
    |x| x + 1
}
```

which also avoids the drawbacks of using a boxed trait object.

Similarly, the concrete types of iterators could become very complex, incorporating the types of all previous iterators in a chain.
Returning `impl Iterator` means that a function only exposes the `Iterator` trait as a bound on its return type, instead of explicitly specifying all of the other iterator types involved.

### Differences between generics and `impl Trait` in return position

In argument position, `impl Trait` is very similar in semantics to a generic type parameter.
However, there are significant differences between the two in return position.
With `impl Trait`, unlike with a generic type parameter, the function chooses the return type, and the caller cannot choose the return type.

The function:

```rust,ignore
fn foo<T: Trait>() -> T {
```

allows the caller to determine the return type, `T`, and the function returns that type.

The function:

```rust,ignore
fn foo() -> impl Trait {
```

doesn't allow the caller to determine the return type.
Instead, the function chooses the return type, but only promises that it will implement `Trait`.

## Limitations

`impl Trait` can only appear as a parameter or return type of a free or inherent function.
It cannot appear inside implementations of traits, nor can it be the type of a let binding or appear inside a type alias.

[closures]: closure.md
[_GenericArgs_]: ../paths.md#paths-in-expressions
[_GenericParams_]: ../items/generics.md
[_TraitBound_]: ../trait-bounds.md
[trait object]: trait-object.md
[_TypeParamBounds_]: ../trait-bounds.md
