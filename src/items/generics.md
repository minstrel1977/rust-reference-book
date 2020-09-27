# 类型参数和生命周期参数

>[generics.md](https://github.com/rust-lang/reference/blob/master/src/items/generics.md)\
>commit f8e76ee9368f498f7f044c719de68c7d95da9972

> **<sup>句法</sup>**\
> _Generics_ :\
> &nbsp;&nbsp; `<` _GenericParams_ `>`
>
> _GenericParams_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _LifetimeParams_\
> &nbsp;&nbsp; | ( _LifetimeParam_ `,` )<sup>\*</sup> _TypeParams_
>
> _LifetimeParams_ :\
> &nbsp;&nbsp; ( _LifetimeParam_ `,` )<sup>\*</sup> _LifetimeParam_<sup>?</sup>
>
> _LifetimeParam_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>?</sup> [LIFETIME_OR_LABEL]&nbsp;( `:` [_LifetimeBounds_] )<sup>?</sup>
>
> _TypeParams_:\
> &nbsp;&nbsp; ( _TypeParam_ `,` )<sup>\*</sup> _TypeParam_<sup>?</sup>
>
> _TypeParam_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>?</sup> [IDENTIFIER] ( `:` [_TypeParamBounds_]<sup>?</sup> )<sup>?</sup> ( `=` [_Type_] )<sup>?</sup>

函数、类型别名、结构体、枚举、联合体、trait 和实现可以通过类型和生命周期达到*参数化*配置的的效果。这些参数在尖括号<span class="parenthetical">（`<…>`）</span>中列出，通常紧跟在数据项名称之后和数据项定义之前。对于没有名称的实现，它们直接位于 `impl` 之后。生命周期参数必须在类型参数之前声明。下面给出一些带有类型参数和生命周期参数的数据项的一些示例：

```rust
fn foo<'a, T>() {}
trait A<U> {}
struct Ref<'a, T> where T: 'a { r: &'a T }
```

[引用]、[裸指针]、[数组]、[切片][数组]、[元组]和[函数指针]也有生命周期或类型参数，但它们不能使用路径语法去引用。

## where子句

> **<sup>句法</sup>**\
> _WhereClause_ :\
> &nbsp;&nbsp; `where` ( _WhereClauseItem_ `,` )<sup>\*</sup> _WhereClauseItem_ <sup>?</sup>
>
> _WhereClauseItem_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _LifetimeWhereClauseItem_\
> &nbsp;&nbsp; | _TypeBoundWhereClauseItem_
>
> _LifetimeWhereClauseItem_ :\
> &nbsp;&nbsp; [_Lifetime_] `:` [_LifetimeBounds_]
>
> _TypeBoundWhereClauseItem_ :\
> &nbsp;&nbsp; _ForLifetimes_<sup>?</sup> [_Type_] `:` [_TypeParamBounds_]<sup>?</sup>
>
> _ForLifetimes_ :\
> &nbsp;&nbsp; `for` `<` [_LifetimeParams_](#type-and-lifetime-parameters) `>`

*where子句*提供了另一种方法来指定类型参数和生命周期参数上的约束，以及一种指定不是类型参数的类型上的约束的方法。

当定义项时，约束与数据项的参数或生命周期（包括高阶生命周期）无关也是可以的。但这样做会带来潜在的错误。（译者注：这句译者也不太理解，但为了保持完整性，先这么翻着，但也打个TobeModify的标记，等以后真懂了再改，同时附上原文：Bounds that don't use the item's parameters or higher-ranked lifetimes are checked when the item is defined. It is an error for such a bound to be false.）

在定义数据项时，还会检查某些泛型类型的 [`Copy`]、[`Clone`] 和 [`Sized`] 约束。将 `Copy` 或 `Clone` 作为可变引用、[trait对象]或[切片][数组]的约束上错误的，将 `Sized` 作为 trait对象或切片的约束也是错误的。
<!-- [`Copy`], [`Clone`], and [`Sized`] bounds are also checked for certain generic types when defining the item. It is an error to have `Copy` or `Clone`as a bound on a mutable reference, [trait object] or [slice][arrays] or `Sized` as a bound on a trait object or slice. -->

```rust,compile_fail
struct A<T>
where
    T: Iterator,            // 可以用 A<T: Iterator> 来替代
    T::Item: Copy,
    String: PartialEq<T>,
    i32: Default,           // 允许，但没什么用
    i32: Iterator,          // 错误: 此 trait约束不适合，i32 没有实现 Iterator
    [T]: Copy,              // 错误: 此 trait约束不适合，切片上不能有此 trait约束
{
    f: T,
}
```

## 属性

泛型生命周期和类型参数允许[属性]，但在目前这个位置还没有任何任何有意义的内置属性，但用户可能可以通过自定义的派生属性来做些有意义的属性设置。

下面示例演示如何使用自定义派生属性修改泛型参数的含义。

<!-- ignore: requires proc macro derive -->
```rust,ignore
// 假设 MyFlexibleClone 的派生将 `my_flexible_clone` 声明为它可以理解的属性。
#[derive(MyFlexibleClone)]
struct Foo<#[my_flexible_clone(unbounded)] H> {
    a: *const H
}
```

[IDENTIFIER]: ../identifiers.md
[LIFETIME_OR_LABEL]: ../tokens.md#生命周期和循环标签

[_LifetimeBounds_]: ../trait-bounds.md
[_Lifetime_]: ../trait-bounds.md
[_OuterAttribute_]: ../attributes.md
[_Type_]: ../types.md#type-expressions
[_TypeParamBounds_]: ../trait-bounds.md

[数组]: ../types/array.md
[函数指针]: ../types/function-pointer.md
[引用]: ../types/pointer.md#shared-references-
[裸指针]: ../types/pointer.md#raw-pointers-const-and-mut
[`Clone`]: ../special-types-and-traits.md#clone
[`Copy`]: ../special-types-and-traits.md#copy
[`Sized`]: ../special-types-and-traits.md#sized
[元组]: ../types/tuple.md
[trait对象]: ../types/trait-object.md
[属性]: ../attributes.md
