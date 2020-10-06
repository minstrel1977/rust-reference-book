# Trait and lifetime bounds
# trait约束 和生存期约束

>[trait-bounds.md](https://github.com/rust-lang/reference/blob/master/src/trait-bounds.md)\
>commit f8e76ee9368f498f7f044c719de68c7d95da9972

> **<sup>句法</sup>**\
> _TypeParamBounds_ :\
> &nbsp;&nbsp; _TypeParamBound_ ( `+` _TypeParamBound_ )<sup>\*</sup> `+`<sup>?</sup>
>
> _TypeParamBound_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _Lifetime_ | _TraitBound_
>
> _TraitBound_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `?`<sup>?</sup>
> [_ForLifetimes_](#higher-ranked-trait-bounds)<sup>?</sup> [_TypePath_]\
> &nbsp;&nbsp; | `(` `?`<sup>?</sup>
> [_ForLifetimes_](#higher-ranked-trait-bounds)<sup>?</sup> [_TypePath_] `)`
>
> _LifetimeBounds_ :\
> &nbsp;&nbsp; ( _Lifetime_ `+` )<sup>\*</sup> _Lifetime_<sup>?</sup>
>
> _Lifetime_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [LIFETIME_OR_LABEL]\
> &nbsp;&nbsp; | `'static`\
> &nbsp;&nbsp; | `'_`

[trait][Trait]约束和生存期约束为[泛型项][generic]提供了一种方法来限制将哪些类型和生存期用作参数。通过 [where子句][where clause]可以为任何类型提供约束。对于某些常见的情况，也可以使用如下简写形式：

* 紧跟在[泛型参数][generic]声明之后的约束：`fn f<A: Copy>() {}` 与 `fn f<A> where A: Copy () {}` 效果一样。
* 在 trait声明中作为指定[超类trait][supertraits] 约束时：`trait Circle : Shape {}` 等同于 `trait Circle where Self : Shape {}`。
* 在trait声明中作为指定关联类型上的约束时：`trait A { type B: Copy; }` 等同于 `trait A where Self::B: Copy { type B; }`。

使用数据项时必须满足该数据项的约束。当对泛型项进行类型检查和借用检查时，约束可用来确认当前准备用来单态化此泛型的实例类型是否实现了约束给出的 trait。例如，给定 `Ty: Trait`
Bounds on an item must be satisfied when using the item. When type checking and borrow checking a generic item, the bounds can be used to determine that a trait is implemented for a type. For example, given `Ty: Trait`

* 在泛型函数体中，`Trait` 中的方法可以被 `Ty`值调用。同样，`Trait` 上的相关常数也可以被使用。
* `Trait` 上的关联类型可以被使用。
* 具有 `T: Trait`约束的泛型函数和类型可以在使用 `T` 的地方使用 `Ty`。

```rust
# type Surface = i32;
trait Shape {
    fn draw(&self, Surface);
    fn name() -> &'static str;
}

fn draw_twice<T: Shape>(surface: Surface, sh: T) {
    sh.draw(surface);           // 能调用此方法上因为 T: Shape
    sh.draw(surface);
}

fn copy_and_draw_twice<T: Copy>(surface: Surface, sh: T) where T: Shape {
    let shape_copy = sh;        // sh 没有 move 是因为 T: Copy
    draw_twice(surface, sh);    // 能使用泛型函数 draw_twice 是因为 T: Shape
}

struct Figure<S: Shape>(S, S);

fn name_figure<U: Shape>(
    figure: Figure<U>,          // 这里类型 Figure<U> 的格式正确是因为 U: Shape
) {
    println!(
        "Figure of two {}",
        U::name(),              // 可以使用关联函数
    );
}
```

trait 和生存期约束也被用来命名 [trait对象][trait objects]。

## `?Sized`

`?` 仅用于声明 [`Sized`] trait可能不会被某类型参数或关联类型实现。`?Sized` 不能用作其他类型的绑定。
<!-- `?` is only used to declare that the [`Sized`] trait may not be implemented for a type parameter or associated type. `?Sized` may not be used as a bound for other types. tobemodify-->

## Lifetime bounds
## 生存期约束

生存期约束可以应用于类型或其他生存期。约束 `'a: 'b` 通常被解读为 *`'a` 比 `'b` 活得久*。`'a: 'b`意味着 `'a` 持续时间比 `'b` 长，所以只要 `&'b ()` 有效，引用 `&'a ()` 就有效。

```rust
fn f<'a, 'b>(x: &'a i32, mut y: &'b i32) where 'a: 'b {
    y = x;                      // 因为 'a: 'b，所以&'a i32 是 &'b i32 的子类型
    let r: &'b &'a i32 = &&0;   // &'b &'a i32 格式合法是因为 'a: 'b
}
```
`T: 'a` 意味着 `T` 的所有生存期参数都比 `'a` 活得长。例如，如果 `'a` 是一个任意的(unconstrained)生存期参数，那么 `i32: 'static` 和 `&'static str: 'a` 合法，但 `Vec<&'a ()>: 'static` 不合法。

## Higher-ranked trait bounds
## 高阶 trait约束

生存期可以被抽象进*高阶*的类型约束。这些约束指定了一个对*所有*生存期都为真的边界。例如，像 `for<'a> &'a T: PartialEq<i32>` 这样的约束需要一个如下的实现
Type bounds may be higher ranked over lifetimes. These bounds specify a bound is true *for all* lifetimes. For example, a bound such as `for<'a> &'a T: PartialEq<i32>` would require an implementation like

```rust
# struct T;
impl<'a> PartialEq<i32> for &'a T {
    // ...
#    fn eq(&self, other: &i32) -> bool {true}
}
```

然后可以用来比较任意生存期的 `&'a T` 和 `i32`。
and could then be used to compare a `&'a T` with any lifetime to an `i32`.

下面这类场景只能使用更高级别的约束，因为引用的生命周期比函数的生命周期参数短
Only a higher-ranked bound can be used here as the lifetime of the reference is shorter than a lifetime parameter on the function:

```rust
fn call_on_ref_zero<F>(f: F) where for<'a> F: Fn(&'a i32) {
    let zero = 0;
    f(&zero);
}
```

高阶生存期也可以在 trait 之前指定，唯一的区别是生存期参数的作用域，它只扩展到下面的 trait 的末尾，而不是整个范围。下面这个函数和上一个等价。
Higher-ranked lifetimes may also be specified just before the trait, the only difference is the scope of the lifetime parameter, which extends only to the end of the following trait instead of the whole bound. This function is equivalent to the last one.

```rust
fn call_on_ref_zero<F>(f: F) where F: for<'a> Fn(&'a i32) {
    let zero = 0;
    f(&zero);
}
```

[LIFETIME_OR_LABEL]: tokens.md#lifetimes-and-loop-labels
[_TypePath_]: paths.md#类型中的路径
[`Sized`]: special-types-and-traits.md#sized

[associated types]: items/associated-items.md#associated-types
[supertraits]: items/traits.md#supertraits
[generic]: items/generics.md
[Trait]: items/traits.md#trait-bounds
[trait objects]: types/trait-object.md
[where clause]: items/generics.md#where子句
