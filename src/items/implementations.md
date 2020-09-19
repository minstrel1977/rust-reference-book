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

固有实现被定义为一段由关键字 `impl`、泛型类型声明、指向标称类型(nominal type)的路径、where子句和一对花括号括起来的一组*关联项(associable items)*组成的序列。

（这里）*标称类型*也被称作*实现类型*；*关联项*可理解为*实现类型*里的各种关联数据项。（译者注：这里译者大致采取了意译。首先 nominal type 在目前国内的Rust 社区没有合适的翻译先例，所以译者这里就蹭机器学习的热点翻译为“标称类型”，这种行径虽然降低了Rust的逼格，但在目前状态下，收益可能还是值得的；其次后半句采取意译是因为aassociable item 和  associable items 该怎么区分翻译，该怎么组织语序实在麻烦，译者又不想再创立新名词，所以只能模糊一下。囿于译者本身水平，翻译可能有错，所以本句原文也附上：The nominal type is called the _implementing type_ and the associable items are the _associated items_ to the implementing type.）

固有实现将包含的数据项与实现类型关联起来。固有实现可以包含[关联函数]（包括方法）和[关联常量]。固有实现不能包含关联的类型别名。

关联数据项的[路径]的前半部是其实现类型的所有（形式的）路径中的任一种，然后再拼接上这个关联数据项的标识符（作为的路径组件）。

类型可以有多个固有实现。实现类型的定义与原始类型的定义必须在同一个 crate 里。

``` rust
pub mod color {
    // 译者添加的注释：这段为 Color类型 或 原始Color类型
    pub struct Color(pub u8, pub u8, pub u8);
    // 译者添加的注释：这里 `Color` 为Color类型的标称类型 或 实现类型
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

    // 实现类型重新导出后，新导出的路径表示也可用。
    Color::red();

    // 这个不行, 因为 `values` 非公有.
    // values::Color::red();
}
```

## trait实现

*trait实现*的定义与固有实现类似，只是可选的泛型类型声明后跟一个 [trait]，再后跟关键字  `for`。后面是一个指向标称类型的路径。

<!-- 为理解这个，你必须回去查看一下上一节的内容 :( -->

trait 即为*trait接口*。实现类型实现了trait接口。

trait实现必须定义trait接口 声明的所有非默认关联数据项，可以重新定义trait接口 定义的默认关联数据项，并且不能定义任何其他数据项。

关联项的完整路径为 `<` 后接实现类型的路径，再后接 `as`，然后是指向 trait 的路径，再后跟 `>`，这整体作为一个路径组件，然后再接关联数据项自己的路径组件。

非安全trait 需要trait实现以关键字 `unsafe` 开头。

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

### trait实现的一致性

如果孤儿规则检查失败或存在重叠的实现实例，则认为 trait实现不一致。

当两个实现各自的 *trait接口*集之间存在非空交集时即为这两个 *trait实现*重叠了，（这种常情况下）可以用相同的类型来实例化这两种不同实现的实例。<!-- 这有可能是错的?因为对于输入类型参数来说，不能用同一组类型去实例化两个实现-->

#### 孤儿规则

给定 `impl<P1..=Pn> Trait<T1..=Tn> for T0`，只有以下至少一种情况为真时，此 `impl` 才能成立：

- `Trait` 是一个[本地 trait]
- 以下所有
  - `T0..=Tn` 种的类型至少有一种是[本地类型]。假设 `Ti` 是第一个开始这样的类型。
  - 没有[无覆盖类型]参数 `P1..=Pn` 会出现在 `T0..Ti` 里（注意 `Ti` 被排除在外）。
  - （译者注：然后以同样的规则再检测 `Ti+1`，以此类推，直到检测完 `Tn`）

只有*无覆盖*类型参数的外观被限制。注意，为了一致性的目的，[基本类型]是特殊的。不认为 `Box<T>` 中的 `T` 是有覆盖的，而 `Box<LocalType>` 有被认为是本地的。
Only the appearance of *uncovered* type parameters is restricted. Note that for the purposes of coherence, [fundamental types] are special. The `T` in `Box<T>` is not considered covered, and `Box<LocalType>` is considered local.


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
[基本类型]: ../glossary.md#fundamental-type-constructors
[无覆盖类型]: ../glossary.md#uncovered-type
