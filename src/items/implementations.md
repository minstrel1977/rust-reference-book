# 实现

>[implementations.md](https://github.com/rust-lang/reference/blob/master/src/items/implementations.md)\
>commit 6a78aa5b0b09f78a6fa28b9dc3078c9b134785a9

> **<sup>句法</sup>**\
> _Implementation_ :\
> &nbsp;&nbsp; _InherentImpl_ | _TraitImpl_
>
> _InherentImpl_ :\
> &nbsp;&nbsp; `impl` [_Generics_]<sup>?</sup>&nbsp;[_Type_]&nbsp;[_WhereClause_]<sup>?</sup> `{`\
> &nbsp;&nbsp; &nbsp;&nbsp; [_InnerAttribute_]<sup>\*</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp; _InherentImplItem_<sup>\*</sup>\
> &nbsp;&nbsp; `}`
>
> _InherentImplItem_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> (\
> &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; [_MacroInvocationSemi_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | ( [_Visibility_]<sup>?</sup> ( [_ConstantItem_] | [_Function_] | [_Method_] ) )\
> &nbsp;&nbsp; )
>
> _TraitImpl_ :\
> &nbsp;&nbsp; `unsafe`<sup>?</sup> `impl` [_Generics_]<sup>?</sup> `!`<sup>?</sup>
>              [_TypePath_] `for` [_Type_]\
> &nbsp;&nbsp; [_WhereClause_]<sup>?</sup>\
> &nbsp;&nbsp; `{`\
> &nbsp;&nbsp; &nbsp;&nbsp; [_InnerAttribute_]<sup>\*</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp; _TraitImplItem_<sup>\*</sup>\
> &nbsp;&nbsp; `}`
>
> _TraitImplItem_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> (\
> &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; [_MacroInvocationSemi_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | ( [_Visibility_]<sup>?</sup> ( [_TypeAlias_] | [_ConstantItem_] | [_Function_] | [_Method_] ) )\
> &nbsp;&nbsp; )

*实现*是将数据项与*实现类型*关联起来的数据项。实现使用关键字`impl` 定义，并包含了属于正要实现的类型的实例的函数，或者包含了正要实现的类型本身的静态函数。

有两种类型的实现:

- 固有实现
- [trait]实现

## 固有实现

固有实现被定义为一段由关键字 `impl`、泛型类型声明、指向标称类型(nominal type)的路径、where子句和一对花括号括起来的一组*关联项(associable items)*组成的序列。（这里）*标称类型*被称为*实现类型*；*关联项*可理解为和（固有实现的）实现类型相关联的各种数据项，（是从描述数据项到实现类型的关系）。（译者注：这里译者大致采取了意译。首先 nominal type 在目前国内的Rust 社区没有合适的翻译先例，所以译者这里就蹭机器学习的热点翻译为“标称类型”，这种行径虽然降低了Rust的逼格，但在目前状态下，收益可能还是值得的；其次后半句采取意译是因为aassociable item 和  associable items 该怎么区分翻译，该怎么组织语序实在麻烦，译者又不想再创立新名词，所以只能模糊一下。囿于译者本身水平，翻译可能有错，所以本句原文也附上：The nominal type is called the _implementing type_ and the associable items are the _associated items_ to the implementing type.）

固有实现将包含的数据项与实现类型关联起来。固有实现可以包含[关联函数]（包括方法）和[关联常量]。固有实现不能包含关联的类型别名。

关联数据项的[路径]的前半部是其实现类型的所有（形式的）路径中的任一种，然后再拼接上这个关联数据项的标识符（作为的路径组件）。

类型可以有多个固有实现。实现类型的定义与原始类型的定义必须在同一个 crate 里。

``` rust
pub mod color {
    // 译者添加的注释：这为 `Color` 的 类型 或 原始类型
    pub struct Color(pub u8, pub u8, pub u8);
    // 译者添加的注释：这为 `Color` 标称类型 或 实现类型
    impl Color {
        pub const WHITE: Color = Color(255, 255, 255);
    }
}

mod values {
    use super::color::Color;
    impl Color {
        pub fn red() -> Color {
            Color(255, 0, 0)
        }
    }
}

pub use self::color::Color;
fn main() {
    // 实现类型 和其 实现 在同一个模块下，此时关联数据项的路径表示。
    color::Color::WHITE;

    // 类型的实现块和类型的声明不在同一个模块下，此时对实现内的关联数据项的存取仍通过指向类型定义的标准模式
    color::Color::red();

    // 重新导出后的实现类型的路径表示仍然可用。
    Color::red();

    // 这个不行, 因为 `values` 非公有.
    // values::Color::red();
}
```

## Trait Implementations

A _trait implementation_ is defined like an inherent implementation except that
the optional generic type declarations is followed by a [trait] followed
by the keyword `for`. Followed by a path to a nominal type.

<!-- To understand this, you have to back-reference to the previous section. :( -->

The trait is known as the _implemented trait_. The implementing type
implements the implemented trait.

A trait implementation must define all non-default associated items declared
by the implemented trait, may redefine default associated items defined by the
implemented trait, and cannot define any other items.

The path to the associated items is `<` followed by a path to the implementing
type followed by `as` followed by a path to the trait followed by `>` as a path
component followed by the associated item's path component.

[Unsafe traits] require the trait implementation to begin with the `unsafe`
keyword.

```rust
# #[derive(Copy, Clone)]
# struct Point {x: f64, y: f64};
# type Surface = i32;
# struct BoundingBox {x: f64, y: f64, width: f64, height: f64};
# trait Shape { fn draw(&self, Surface); fn bounding_box(&self) -> BoundingBox; }
# fn do_draw_circle(s: Surface, c: Circle) { }
struct Circle {
    radius: f64,
    center: Point,
}

impl Copy for Circle {}

impl Clone for Circle {
    fn clone(&self) -> Circle { *self }
}

impl Shape for Circle {
    fn draw(&self, s: Surface) { do_draw_circle(s, *self); }
    fn bounding_box(&self) -> BoundingBox {
        let r = self.radius;
        BoundingBox {
            x: self.center.x - r,
            y: self.center.y - r,
            width: 2.0 * r,
            height: 2.0 * r,
        }
    }
}
```

### Trait Implementation Coherence

A trait implementation is considered incoherent if either the orphan rules check fails
or there are overlapping implementation instances.

Two trait implementations overlap when there is a non-empty intersection of the
traits the implementation is for, the implementations can be instantiated with
the same type. <!-- This is probably wrong? Source: No two implementations can
be instantiable with the same set of types for the input type parameters. -->

#### Orphan rules

Given `impl<P1..=Pn> Trait<T1..=Tn> for T0`, an `impl` is valid only if at
least one of the following is true:

- `Trait` is a [local trait]
- All of
  - At least one of the types `T0..=Tn` must be a [local type]. Let `Ti` be the
    first such type.
  - No [uncovered type] parameters `P1..=Pn` may appear in `T0..Ti` (excluding
    `Ti`)

Only the appearance of *uncovered* type parameters is restricted.
Note that for the purposes of coherence, [fundamental types] are
special. The `T` in `Box<T>` is not considered covered, and `Box<LocalType>` 
is considered local.


## Generic Implementations

An implementation can take type and lifetime parameters, which can be used in
the rest of the implementation. Type parameters declared for an implementation
must be used at least once in either the trait or the implementing type of an
implementation. Implementation parameters are written directly after the `impl`
keyword.

```rust
# trait Seq<T> { fn dummy(&self, _: T) { } }
impl<T> Seq<T> for Vec<T> {
    /* ... */
}
impl Seq<bool> for u32 {
    /* Treat the integer as a sequence of bits */
}
```

## Attributes on Implementations

Implementations may contain outer [attributes] before the `impl` keyword and
inner [attributes] inside the brackets that contain the associated items. Inner
attributes must come before any associated items. That attributes that have
meaning here are [`cfg`], [`deprecated`], [`doc`], and [the lint check
attributes].

[_ConstantItem_]: constant-items.md
[_Function_]: functions.md
[_Generics_]: generics.md
[_InnerAttribute_]: ../attributes.md
[_MacroInvocationSemi_]: ../macros.md#macro-invocation
[_Method_]: associated-items.md#methods
[_OuterAttribute_]: ../attributes.md
[_TypeAlias_]: type-aliases.md
[_TypePath_]: ../paths.md#paths-in-types
[_Type_]: ../types.md#type-expressions
[_Visibility_]: ../visibility-and-privacy.md
[_WhereClause_]: generics.md#where-clauses
[trait]: traits.md
[associated functions]: associated-items.md#associated-functions-and-methods
[associated constants]: associated-items.md#associated-constants
[attributes]: ../attributes.md
[`cfg`]: ../conditional-compilation.md
[`deprecated`]: ../attributes/diagnostics.md#the-deprecated-attribute
[`doc`]: ../../rustdoc/the-doc-attribute.html
[path]: ../paths.md
[the lint check attributes]: ../attributes/diagnostics.md#lint-check-attributes
[Unsafe traits]: traits.md#unsafe-traits
[local trait]: ../glossary.md#local-trait
[local type]: ../glossary.md#local-type
[fundamental types]: ../glossary.md#fundamental-type-constructors
[uncovered type]: ../glossary.md#uncovered-type
