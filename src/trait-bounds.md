# Trait and lifetime bounds
# trait约束和生存期约束

>[trait-bounds.md](https://github.com/rust-lang/reference/blob/master/src/trait-bounds.md)\
>commit: 82517788e162791dad3305a8c8d8d20d49510ad6 \
>本章译文最后维护日期：2024-06-15

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

[trait][Trait]约束和生存期约束为[泛型程序项][generic]提供了一种方法来限制将哪些类型和生存期可被用作它们的参数。通过 [where子句][where clause]可以为任何泛型提供约束。对于某些常见的情况，也可以使用如下简写形式：

* 跟在[泛型参数][generic]声明之后的约束：`fn f<A: Copy>() {}` 与 `fn f<A>() where A: Copy  {}` 效果等价。
* 在 trait声明中作为指定[超类trait(supertraits)][supertraits] 约束时：`trait Circle : Shape {}` 等同于 `trait Circle where Self : Shape {}`。
* 在 trait声明中作为指定关联类型上的约束时：`trait A { type B: Copy; }` 等同于 `trait A where Self::B: Copy { type B; }`。

在程序项上应用了约束就要求在使用该程序项时使用者必须满足这些约束。当对泛型程序项进行类型检查和借用检查时，约束可用来确认当前准备用来单态化此泛型的实例类型是否实现了约束给出的 trait。例如，给定 `Ty: Trait`：

* 在泛型函数体中，`Trait` 中的方法可以被 `Ty`类型的值调用。同样，`Trait` 上的相关常数也可以被使用。
* `Trait` 上的关联类型可以被使用。
* 带有 `T: Trait`约束的泛型函数或类型可以在使用 `T` 的地方替换使用 `Ty`。

```rust
# type Surface = i32;
trait Shape {
    fn draw(&self, surface: Surface);
    fn name() -> &'static str;
}

fn draw_twice<T: Shape>(surface: Surface, sh: T) {
    sh.draw(surface);           // 能调用此方法上因为 T: Shape
    sh.draw(surface);
}

fn copy_and_draw_twice<T: Copy>(surface: Surface, sh: T) where T: Shape {
    let shape_copy = sh;        // sh 没有被使用移动语义移走，是因为 T: Copy
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

`?` 仅用于放宽[类型参数][type parameters]或[关联类型][associated types] 上的[`Sized`] trait。
目前 `?Sized` 还不能用作其他类型的约束。

## Lifetime bounds
## 生存期约束

生存期约束可以应用于类型或其他生存期。
约束 `'a: 'b` 通常被读做 *`'a` 比 `'b` 存活的时间久(outlives)*。
`'a: 'b` 意味着 `'a` 持续的时间至少和 `'b` 一样长，所以只要 `&'a ()` 有效，则 `&'b ()` 必定有效。[^译注1]

```rust
fn f<'a, 'b>(x: &'a i32, mut y: &'b i32) where 'a: 'b {
    y = x;                      // 因为 'a: 'b，所以&'a i32 是 &'b i32 的子类型
    let r: &'b &'a i32 = &&0;   // &'b &'a i32 格式合法是因为 'a: 'b
}
```

`T: 'a` 意味着 `T` 的所有生存期参数都比 `'a` 存活得时间长。
例如，如果 `'a` 是一个任意的(unconstrained)生存期参数，那么 `i32: 'static` 和 `&'static str: 'a` 都合法，但 `Vec<&'a ()>: 'static` 不合法。

## Higher-ranked trait bounds
## 高阶trait约束

> _ForLifetimes_ :\
> &nbsp;&nbsp; `for` [_GenericParams_]

可以在生存期上再进行*更高阶的*类型约束。这些高阶约束指定了一个对*所有*生存期都必须保证为真的约束。例如，像 `for<'a> &'a T: PartialEq<i32>` 这样的约束需要一个如下的实现

```rust
# struct T;
impl<'a> PartialEq<i32> for &'a T {
    // ...
#    fn eq(&self, other: &i32) -> bool {true}
}
```

这样就可以拿任意生存期的 `&'a T` 和 `i32` 做比较啦。

下面这类场景只能使用高阶trait约束，因为引用的生命周期比函数的生命周期参数短：[^译注3]

```rust
fn call_on_ref_zero<F>(f: F) where for<'a> F: Fn(&'a i32) { 
    let zero = 0;
    f(&zero);}
```

>译者注：译者下面举例代码可以和上面原文的代码对比着看。下面代码中，因为 `F` 没约束 `'a`，导致参数 `f` 引用了未经扩展生存期的 `zero`
>```rust,compile_fail 
>fn call_on_ref_zero<'a, F>(f: F) where F: Fn(&'a i32) {
>    let zero = 0;
>    f(&zero);
>}
>```

高阶生存期也可以在 trait 前面指定：唯一的区别是生存期参数的[作用域][hrtb-scopes] ，像下面这样 `'a` 的作用域只扩展到后面跟的 trait 的末尾，而不是整个约束[^译注4]。下面这个函数和上一个等价。

```rust
fn call_on_ref_zero<F>(f: F) where F: for<'a> Fn(&'a i32) {
    let zero = 0;
    f(&zero);
}
```


## Implied bounds
## 隐式约束

生存期约束有效需要以类型定义格式合法(well-formed)为前提，有时甚至需要一些编译器推断。

```rust
fn requires_t_outlives_a<'a, T>(x: &'a T) {}
```
类型参数`T` 需要比类型`&'a T` 的`'a`生存时间长才能形成格式合法(well-formed)。
之所以推断出这一点，是因为函数签名包含类型`&'a T`，该类型仅在 `T: 'a` 成立时才有效。

Rust 会为函数的所有参数和输出添加隐式约束。在 `requires_t_outlives_a`内部，即使没有明确指定，也可以假定 `T: 'a` 得到了保持：

```rust
fn requires_t_outlives_a_not_implied<'a, T: 'a>() {}

fn requires_t_outlives_a<'a, T>(x: &'a T) {
    // 这能够编译是因为 `T: 'a` 是由引用类型 `&'a T` 隐式约束的。
    requires_t_outlives_a_not_implied::<'a, T>();
}
```

```rust,compile_fail,E0309
# fn requires_t_outlives_a_not_implied<'a, T: 'a>() {}
fn not_implied<'a, T>() {
    // 这里报错是因为函数签名并没有 `T: 'a` 的隐式约束。
    requires_t_outlives_a_not_implied::<'a, T>();
}
```

只有生存期约束是隐式约束，特征约束仍然必须显式添加。
因此，以下示例会导致错误：

```rust,compile_fail,E0277
use std::fmt::Debug;
struct IsDebug<T: Debug>(T);
// error[E0277]: `T` 没有实现 `Debug`
fn doesnt_specify_t_debug<T>(x: IsDebug<T>) {}
```

还推断出任何类型的类型定义和impl块的生存期界限：
在类型定义和类型的 impl块中，生存期约束的推断逻辑仍会被编译器执行：

```rust
struct Struct<'a, T> {
    // 这里要求 `T: 'a`， 这点是编译器推断出的
    field: &'a T,
}

enum Enum<'a, T> {
    // 这里要求 `T: 'a`， 这点是编译器推断出的
    //
    // 注意，即便是只使用 `Enum::OtherVariant`，仍需要保持 `T: 'a`
    SomeVariant(&'a T),
    OtherVariant,
}

trait Trait<'a, T: 'a> {}

// 这里会报错是因为任何类型的 impl头中都没有隐含 T: 'a` 的逻辑
//     impl<'a, T> Trait<'a, T> for () {}

// 这里能通过编译是因为 self类型 `&'a T` 中隐含了 `T: 'a`的逻辑
impl<'a, T> Trait<'a, T> for &'a T {}
```


[^译注1]: 译者理解：理解这种关系时，可以把生存期 `'a` 和 `'b` 理解成去引用对象时需传入的参数，给定 `'a: 'b` 和类型 `T`，如果 `'a T`有效，那此时再传入 `'b` 就去引用 `T` 必定有效。

[^译注2]: 译者理解：高阶 trait约束就是对带生存期的类型重新进行约束。像这句中的例子就是对 `&'a T` 加上了 `PartialEq<i32>` 的约束，其中 `for<'a>` 可以理解为：对于 `'a` 的所有可能选择。更多信息请参见：https://doc.rust-lang.org/std/cmp/trait.PartialEq.html 和 https://doc.rust-lang.org/nightly/nomicon/hrtb.html

[^译注3]: 译者理解此例中的代码 `for<'a> F: Fn(&'a i32)` 为：`F` 对于 `'a` 的所有可能选择都受 `Fn(&'a i32)` 的约束。

[^译注4]: 译者理解这句的意思是：如果 `F` 的约束有多个 trait，那这种方式里， `'a` 的作用域只是扩展它后面紧跟的那个 trait 的方法，即 `Fn(&'a i32)` 里。

[LIFETIME_OR_LABEL]: tokens.md#lifetimes-and-loop-labels
[_GenericParams_]: items/generics.md
[_TypePath_]: paths.md#paths-in-types
[`Clone`]: special-types-and-traits.md#clone
[`Copy`]: special-types-and-traits.md#copy
[`Sized`]: special-types-and-traits.md#sized
[arrays]: types/array.md
[associated types]: items/associated-items.md#associated-types
[hrtb-scopes]: names/scopes.md#higher-ranked-trait-bound-scopes
[supertraits]: items/traits.md#supertraits
[generic]: items/generics.md
[higher-ranked lifetimes]: #higher-ranked-trait-bounds
[slice]: types/slice.md
[Trait]: items/traits.md#trait-bounds
[trait object]: types/trait-object.md
[trait objects]: types/trait-object.md
[type parameters]: types/parameters.md
[where clause]: items/generics.md#where-clauses
