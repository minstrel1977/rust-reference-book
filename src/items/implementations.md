# Implementations
# 实现

>[implementations.md](https://github.com/rust-lang/reference/blob/master/src/items/implementations.md)\
>commit: b2d11240bd9a3a6dd34419d0b0ba74617b23d77e \
>本译文最后维护日期：2020-10-21

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

*实现*是将数据项与*实现类型(implementing type)*关联起来的数据项。实现使用关键字 `impl` 定义，它包含了属于当前实现的类型的实例的函数，或者包含了当前实现的类型本身的静态函数。（译者注：后一句如果简单意译应该是：实现使用关键字 `impl` 定义，它包含了当前类型的方法和函数）

有两种类型的实现:

- 固有实现(inherent implementations)
- [trait]实现(trait implementations)

## Inherent Implementations
## 固有实现

固有实现被定义为一段由关键字 `impl`，泛型类型声明，指向标称类型(nominal type)的路径，一个 where子句和一对花括号括起来的一组*类型关联项(associable items)*组成的序列。

（这里）*标称类型*也被称作*实现类型(implementing type)*；*类型关联项(associable items)*可理解为实现类型的各种*关联数据项*。（译者注：这里译者大致采取了意译。首先 nominal type 在目前国内 Rust 社区里还没有合适的翻译先例，所以译者这里就蹭机器学习的热点翻译为“标称类型”，这种行径虽然降低了Rust的逼格，但在目前状态下，收益可能还是值得的。）

固有实现将包含的数据项与实现类型关联起来。固有实现可以包含[关联函数][associated functions]（包括方法）和[关联常量][associated constants]。固有实现不能包含关联类型别名。

关联数据项的[路径][path]是其实现类型的所有（形式的）路径中的任一种，然后再拼接上这个关联数据项的标识符来作为整个路径的末段路径组件(final path component)。

类型可以有多个固有实现。但作为原始类型定义的实现类型与这些固有实现必须在同一个 crate 里。

``` rust
pub mod color {
    // 译者添加的注释：这段为 Color的原始类型，也是后面实现的实现类型
    pub struct Color(pub u8, pub u8, pub u8);
    // 译者添加的注释：下行的 `Color` 为标称类型 或 实现类型
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
    // 实现类型 和 固有实现 在同一个模块下。
    color::Color::WHITE;

    // 固有实现和类型声明不在同一个模块下，此时对固有实现内的关联数据项的存取仍通过指向实现类型的路径
    color::Color::red();

    // 实现类型重导出后，使用这类快捷路径效果也一样。
    Color::red();

    // 这个不行, 因为 `values` 非公有。
    // values::Color::red();
}
```

## Trait Implementations
## trait实现

*trait实现*的定义与固有实现类似，只是可选的泛型类型声明后须跟一个 [trait]，再后跟关键字 `for`，之后再跟那个指向标称类型的路径。

<!-- 为理解这个，你必须回去查看一下上一节的内容 :( -->

这里讨论的 trait 也被称为*应实现trait(implemented trait)*。实现类型会实现 该应实现trait。

trait实现必须定义应实现trait 声明的所有非默认关联数据项，可以重新定义应实现trait 定义的默认关联数据项，但不能定义任何其他数据项。

关联数据项的完整路径为 `<` 后跟实现类型的路径，再后跟 `as`，然后是指向 trait 的路径，再后跟 `>`，这整体作为一个路径组件，然后再接关联数据项自己的路径组件。

非安全trait 需要 trait实现以关键字 `unsafe` 开头。

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
### trait实现的一致性

一个trait实现如果通不过孤儿规则检查或有其它 trait实现和本 trait实现有重叠，则认为 trait实现不一致(incoherent)。

当两个实现各自的 *trait接口*集之间存在非空交集时即为这两个 trait实现 重叠了，（这种常情况下）这两个 trait实现可以用相同的类型来实例化。<!-- 这可能是错的？来源：对于输入类型参数，没有两个实现可以用相同的类型集实例化。 -->

#### Orphan rules
#### 孤儿规则

给定 `impl<P1..=Pn> Trait<T1..=Tn> for T0`，只有以下至少一种情况为真时，此 `impl` 才能成立：

- `Trait` 是一个[本地trait][local trait]
- 以下所有
  - `T0..=Tn` 中的类型至少有一种是[本地类型][local type]。假设 `Ti` 是第一个这样的类型。
  - 没有[无覆盖类型][uncovered type]参数 `P1..=Pn` 会出现在 `T0..Ti` 里（注意 `Ti` 被排除在外）。
  （译者注：为理解上面两条规则，举几个例子、 `impl<T> ForeignTrait<LocalType> for ForeignType<T>` 这样的实现也是被允许的，而 `impl<Vec<T>> ForeignTrait<LocalType> for ForeignType<T>` 和 `impl<T> ForeignTrait<Vec<T>> for ForeignType<T>` 不被允许。）

这里只有*无覆盖*类型参数的外观被限制（译者注：为方便理解“无覆盖类型参数”，译者提醒读者把它想象成上面译者举例中的 `T`）。注意，理解“无覆盖类型参数”时需要注意：为了保持一致性，[基本类型][fundamental types]虽然外观形式特殊，但仍不认为是有覆盖的，比如 `Box<T>` 中的 `T` 就不认为是有覆盖的，`Box<LocalType>` 这样的就被认为是本地的。

## Generic Implementations
## 泛型实现

实现可以采用类型和生存期作为参数，这些参数可用在实现中（，也可以不用）。为实现所声明的类型参数必须在 trait 中或实现类型中至少使用一次。实现的参数直接写在关键字 `impl` 之后。

```rust
# trait Seq<T> { fn dummy(&self, _: T) { } }
impl<T> Seq<T> for Vec<T> {
    /* ... */
}
impl Seq<bool> for u32 {
    /* 将整数处理为 bits 序列 */
}
```

## Attributes on Implementations
## 实现上的属性

实现可以在关键字 `impl` 之前引入外部[属性][attributes]，在代码体内引入内部[属性][attributes]。内部属性必须位于任何关联数据项之前。这里有意义的属性有 [`cfg`]、[`deprecated`]、[`doc`] 和 [lint检查类属性][the lint check attributes]。

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
[`doc`]: https://doc.rust-lang.org/rustdoc/the-doc-attribute.html
[path]: ../paths.md
[the lint check attributes]: ../attributes/diagnostics.md#lint-check-attributes
[Unsafe traits]: traits.md#unsafe-traits
[local trait]: ../glossary.md#local-trait
[local type]: ../glossary.md#local-type
[fundamental types]: ../glossary.md#fundamental-type-constructors
[uncovered type]: ../glossary.md#uncovered-type

<!-- 2020-11-3 -->
<!-- checked -->
