# Type aliases
# 类型别名

>[type-aliases.md](https://github.com/rust-lang/reference/blob/master/src/items/type-aliases.md)\
>commit: 761ad774fcb300f2b506fed7b4dbe753cda88d80 \
>本章译文最后维护日期：2021-1-17

> **<sup>句法</sup>**\
> _TypeAlias_ :\
> &nbsp;&nbsp; `type` [IDENTIFIER]&nbsp;[_GenericParams_]<sup>?</sup>
>              [_WhereClause_]<sup>?</sup> ( `=` [_Type_] ) `;`

*类型别名*为现有的[类型][type]定义一个新名称。类型别名用关键字 `type` 声明。每个值都是一个唯一的特定的类型，但是可以实现几个不同的 trait，或者兼容几个不同的类型约束。

例如，下面将类型 `Point` 定义为类型 `(u8, u8)` 的同义词/别名：

```rust
type Point = (u8, u8);
let p: Point = (41, 68);
```

元组结构体或单元结构体的类型别名不能用于充当该类型的构造函数:

```rust,edition2018,compile_fail
struct MyStruct(u32);

use MyStruct as UseAlias;
type TypeAlias = MyStruct;

let _ = UseAlias(5); // OK
let _ = TypeAlias(5); // 不能正常执行
```

没有指定代表[类型][_Type_]的类型别名只能作为 [trait] 中的[关联类型][associated type]出现。

[IDENTIFIER]: ../identifiers.md
[_GenericParams_]: generics.md
[_WhereClause_]: generics.md#where-clauses
[_Type_]: ../types.md#type-expressions
[associated type]: associated-items.md#associated-types
[trait]: traits.md
[type]: ../types.md


<!-- 2021-1-17-->
<!-- checked -->
