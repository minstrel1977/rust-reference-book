# Type and Lifetime Parameters
# 类型参数和生存期参数

>[generics.md](https://github.com/rust-lang/reference/blob/master/src/items/generics.md)\
>commit: f8e76ee9368f498f7f044c719de68c7d95da9972 \
>本章译文最后维护日期：2020-11-9

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

函数、类型别名、结构体、枚举、联合体、trait 和实现可以通过类型参数和生存期参数达到*参数化*配置的的效果。这些参数在尖括号<span class="parenthetical">（`<…>`）</span>中列出，通常都是紧跟在程序项名称之后和程序项的定义之前。对于实现，因为它没有名称，那它们就直接位于关键字 `impl` 之后。生存期参数必须在类型参数之前声明。下面给出一些带类型参数和生存期参数的程序项的示例：

```rust
fn foo<'a, T>() {}
trait A<U> {}
struct Ref<'a, T> where T: 'a { r: &'a T }
```

[引用][References]、[裸指针][raw pointers]、[数组][arrays]、[切片][arrays]、[元组][tuples]和[函数指针][function pointers]也有生存期参数或类型参数，但这些程序项不能使用路径句法去引用。

## Where clauses
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

*where子句*提供了另一种方法来为类型参数和生存期参数指定约束(bound)，甚至可以为非类型参数的类型指定约束。

当定义程序项时，where子句提供的约束与该程序项的各种参数（包括生存期和高阶生存期）无关也是可以通过安全检查的。但这样做会带来潜在的错误。

对某些泛型类型来说，在定义它的 where子句时，[`Copy`]、[`Clone`] 和 [`Sized`] 这些约束也可以通过安全检查。但将 `Copy` 或 `Clone` 作为可变引用、[trait对象][trait object]或[切片][arrays]的约束上错误的，将 `Sized` 作为 trait对象或切片的约束也是错误的。

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

## Attributes
## 属性

泛型生存期参数和泛型类型参数允许[属性][attributes]，但在目前这个位置还没有任何任何有意义的内置属性，但用户可能可以通过自定义的派生属性来设置一些有意义的属性。

下面示例演示如何使用自定义派生属性修改泛型参数的含义。

<!-- ignore: requires proc macro derive -->
```rust,ignore
// 假设 MyFlexibleClone 的派生项将 `my_flexible_clone` 声明为它可以理解的属性。
#[derive(MyFlexibleClone)]
struct Foo<#[my_flexible_clone(unbounded)] H> {
    a: *const H
}
```

[IDENTIFIER]: ../identifiers.md
[LIFETIME_OR_LABEL]: ../tokens.md#lifetimes-and-loop-labels

[_LifetimeBounds_]: ../trait-bounds.md
[_Lifetime_]: ../trait-bounds.md
[_OuterAttribute_]: ../attributes.md
[_Type_]: ../types.md#type-expressions
[_TypeParamBounds_]: ../trait-bounds.md

[arrays]: ../types/array.md
[function pointers]: ../types/function-pointer.md
[references]: ../types/pointer.md#shared-references-
[raw pointers]: ../types/pointer.md#raw-pointers-const-and-mut
[`Clone`]: ../special-types-and-traits.md#clone
[`Copy`]: ../special-types-and-traits.md#copy
[`Sized`]: ../special-types-and-traits.md#sized
[tuples]: ../types/tuple.md
[trait object]: ../types/trait-object.md
[attributes]: ../attributes.md

<!-- 2020-11-12-->
<!-- checked -->
