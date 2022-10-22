# Type aliases
# 类型别名

>[type-aliases.md](https://github.com/rust-lang/reference/blob/master/src/items/type-aliases.md)\
>commit: 3c8acda52ffcf5f7ca410c76f4fddf0e129592e2 \
>本章译文最后维护日期：2022-10-22

> **<sup>句法</sup>**\
> _TypeAlias_ :\
> &nbsp;&nbsp; `type` [IDENTIFIER]&nbsp;[_GenericParams_]<sup>?</sup>
>              ( `:` [_TypeParamBounds_] )<sup>?</sup>
>              [_WhereClause_]<sup>?</sup> ( `=` [_Type_] [_WhereClause_]<sup>?</sup>)<sup>?</sup> `;`

*类型别名*为现有的[类型][type]定义一个新名称。类型别名用关键字 `type` 声明。每个值都是一个唯一的特定的类型，但是可以实现几个不同的 trait，或者兼容几个不同的类型约束。

例如，下面将类型 `Point` 定义为类型 `(u8, u8)` 的同义词/别名：

```rust
type Point = (u8, u8);
let p: Point = (41, 68);
```

元组结构体或单元结构体的类型别名不能用于充当该类型的构造函数:

```rust,compile_fail
struct MyStruct(u32);

use MyStruct as UseAlias;
type TypeAlias = MyStruct;

let _ = UseAlias(5); // OK
let _ = TypeAlias(5); // 无法工作
```

类型别名不用作关联类型时，必须包含[_type_]规范，而不能包含[_TypeParamBounds_]规范。

类型别名用作 [trait] 中的[关联类型][associated type]时，不能包含[_type_]规范，但可以包括[_TypeParamBounds_]规范。

类型别名用作 [trait impl] 中的[关联类型][associated type]时，必须包含[_type_]规范，而不能包含[_TypeParamBounds_]规范。

[trait impl] 中类型别名上等号前的 Where子句（如 `type TypeAlias<T> where T: Foo = Bar<T>`）已被弃用。目前推荐使用等号后面的 Where子句（如`type TypeAlias<T> = Bar<T> where T: Foo`）。

[IDENTIFIER]: ../identifiers.md
[_GenericParams_]: generics.md
[_TypeParamBounds_]: ../trait-bounds.md
[_WhereClause_]: generics.md#where-clauses
[_Type_]: ../types.md#type-expressions
[associated type]: associated-items.md#associated-types
[trait]: traits.md
[type]: ../types.md
[trait impl]: implementations.md#trait-implementations