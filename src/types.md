{{#include types-redirect.html}}
# Types
# 类型

>[types.md](https://github.com/rust-lang/reference/blob/master/src/types.md)\
>commit af1cf6d3ca3b7a8c434c142148742aa912e37c34

Rust 程序中的每个变量、数据项和值都有一个类型。*值*的*类型*定义了保存它的内存的解释信息以及可能对该值执行的操作。

内置的类型回忆非平凡的方式(in nontrivial ways)紧密地集成到语言中，这种方式是不可能在用户定义的类型中模拟的。用户定义的类型功能有限。

Rust 类型分类列表为：

* 原生类型(primitive types):
    * [布尔型(Boolean)] — `true` 或 `false`
    * [数字型(Numeric)] — 整型(integer) 和 浮点型(float)
    * [文本型(Textual)] — `char` 和 `str`
    * [Never] — `!` — 没有值的类型
*  序列类型(sequence types)：
    * [元组(Tuple)]
    * [数组(Array)]
    * [切片(Slice)]
* 用户自定义类型(user-defined types)：
    * [结构体(Struct)]
    * [枚举(Enum)]
    * [联合体(Union)]
* 函数类型(function types)：
    * [函数(Functions)]
    * [闭包(Closures)]
* 指针类型(pointer types)：
    * [引用(References)]
    * [裸指针(Raw pointers)]
    * [函数指针(Function pointers)]
* trait类型(Trait types):
    * [trait对象(Trait objects)]
    * [实现对象(Impl trait)]

## Type expressions
## 类型表达式

> **<sup>句法</sup>**\
> _Type_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _TypeNoBounds_\
> &nbsp;&nbsp; | [_ImplTraitType_]\
> &nbsp;&nbsp; | [_TraitObjectType_]
>
> _TypeNoBounds_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_ParenthesizedType_]\
> &nbsp;&nbsp; | [_ImplTraitTypeOneBound_]\
> &nbsp;&nbsp; | [_TraitObjectTypeOneBound_]\
> &nbsp;&nbsp; | [_TypePath_]\
> &nbsp;&nbsp; | [_TupleType_]\
> &nbsp;&nbsp; | [_NeverType_]\
> &nbsp;&nbsp; | [_RawPointerType_]\
> &nbsp;&nbsp; | [_ReferenceType_]\
> &nbsp;&nbsp; | [_ArrayType_]\
> &nbsp;&nbsp; | [_SliceType_]\
> &nbsp;&nbsp; | [_InferredType_]\
> &nbsp;&nbsp; | [_QualifiedPathInType_]\
> &nbsp;&nbsp; | [_BareFunctionType_]\
> &nbsp;&nbsp; | [_MacroInvocation_]

上面*类型*语法规则中定义的*类型表达式*是引用类型的句法。它可以指向：

* 序列类型 ([tuple], [array], [slice]).
* [类型路径(type paths)] 可指：
    * 原生类型([boolean], [numeric], [textual]).
    * Paths to an [item] ([struct], [enum], [union], [type alias], [trait]).
    * [`Self` path] where `Self` is the implementing type.
    * Generic [type parameters].
* Pointer types ([reference], [raw pointer], [function pointer]).
* The [inferred type] which asks the compiler to determine the type.
* [Parentheses] which are used for disambiguation.
* Trait types: [Trait objects] and [impl trait].
* The [never] type.
* [Macros] which expand to a type expression.

### Parenthesized types

> _ParenthesizedType_ :\
> &nbsp;&nbsp; `(` [_Type_] `)`

In some situations the combination of types may be ambiguous. Use parentheses
around a type to avoid ambiguity. For example, the `+` operator for [type
boundaries] within a [reference type] is unclear where the
boundary applies, so the use of parentheses is required. Grammar rules that
require this disambiguation use the [_TypeNoBounds_] rule instead of
[_Type_].

```rust
# use std::any::Any;
type T<'a> = &'a (dyn Any + Send);
```

## Recursive types

Nominal types &mdash; [structs], [enumerations], and [unions] &mdash; may be
recursive. That is, each `enum` variant or `struct` or `union` field may
refer, directly or indirectly, to the enclosing `enum` or `struct` type
itself. Such recursion has restrictions:

* Recursive types must include a nominal type in the recursion (not mere [type
  aliases], or other structural types such as [arrays] or [tuples]). So `type
  Rec = &'static [Rec]` is not allowed.
* The size of a recursive type must be finite; in other words the recursive
  fields of the type must be [pointer types].
* Recursive type definitions can cross module boundaries, but not module
  *visibility* boundaries, or crate boundaries (in order to simplify the module
  system and type checker).

An example of a *recursive* type and its use:

```rust
enum List<T> {
    Nil,
    Cons(T, Box<List<T>>)
}

let a: List<i32> = List::Cons(7, Box::new(List::Cons(13, Box::new(List::Nil))));
```

[_ArrayType_]: types/array.md
[_BareFunctionType_]: types/function-pointer.md
[_ImplTraitTypeOneBound_]: types/impl-trait.md
[_ImplTraitType_]: types/impl-trait.md
[_InferredType_]: types/inferred.md
[_MacroInvocation_]: macros.md#宏调用
[_NeverType_]: types/never.md
[_ParenthesizedType_]: types.md#parenthesized-types
[_QualifiedPathInType_]: paths.md#限定路径
[_RawPointerType_]: types/pointer.md#raw-pointers-const-and-mut
[_ReferenceType_]: types/pointer.md#shared-references-
[_SliceType_]: types/slice.md
[_TraitObjectTypeOneBound_]: types/trait-object.md
[_TraitObjectType_]: types/trait-object.md
[_TupleType_]: types/tuple.md#tuple-types
[_TypeNoBounds_]: types.md#type-expressions
[_TypePath_]: paths.md#类型中的路径
[_Type_]: types.md#type-expressions

[Array]: types/array.md
[Boolean]: types/boolean.md
[Closures]: types/closure.md
[Enum]: types/enum.md
[Function pointers]: types/function-pointer.md
[Functions]: types/function-item.md
[Impl trait]: types/impl-trait.md
[Macros]: macros.md
[Numeric]: types/numeric.md
[Parentheses]: #parenthesized-types
[Raw pointers]: types/pointer.md#raw-pointers-const-and-mut
[References]: types/pointer.md#shared-references-
[Slice]: types/slice.md
[Struct]: types/struct.md
[Textual]: types/textual.md
[Trait objects]: types/trait-object.md
[Tuple]: types/tuple.md
[Type paths]: paths.md#类型中的路径
[Union]: types/union.md
[`Self` path]: paths.md#self-1
[arrays]: types/array.md
[enumerations]: types/enum.md
[function pointer]: types/function-pointer.md
[inferred type]: types/inferred.md
[item]: items.md
[never]: types/never.md
[pointer types]: types/pointer.md
[raw pointer]: types/pointer.md#raw-pointers-const-and-mut
[reference type]: types/pointer.md#shared-references-
[reference]: types/pointer.md#shared-references-
[structs]: types/struct.md
[trait]: types/trait-object.md
[tuples]: types/tuple.md
[type alias]: items/type-aliases.md
[type aliases]: items/type-aliases.md
[type boundaries]: trait-bounds.md
[type parameters]: types/parameters.md
[unions]: types/union.md
