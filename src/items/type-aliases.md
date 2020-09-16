# 类型别名

>[type-aliases.md](https://github.com/rust-lang/reference/blob/master/src/items/type-aliases.md)\
>commit d150e88973ffccc4439111d8e1b26da745670fa8

> **<sup>句法</sup>**\
> _TypeAlias_ :\
> &nbsp;&nbsp; `type` [IDENTIFIER]&nbsp;[_Generics_]<sup>?</sup>
>              [_WhereClause_]<sup>?</sup> `=` [_Type_] `;`

*类型别名*为现有的[类型]定义一个新名称。类型别名用关键字 `type` 声明。每个值都有一个特定的类型，但是可以实现几个不同的 trait，或者兼容几个不同的类型约束。

[类型]: ../types.md

例如，下面将类型 `Point` 定义为类型 `(u8, u8)` 的同义词，类型为 8 位无符号整数对：

```rust
type Point = (u8, u8);
let p: Point = (41, 68);
```

元组结构体或单元结构体的类型别名不能用于限定该类型的构造函数:

```rust,edition2018,compile_fail
struct MyStruct(u32);

use MyStruct as UseAlias;
type TypeAlias = MyStruct;

let _ = UseAlias(5); // OK
let _ = TypeAlias(5); // Doesn't work
```

[IDENTIFIER]: ../identifiers.md
[_Generics_]: generics.md
[_WhereClause_]: generics.md#where-clauses
[_Type_]: ../types.md#type-expressions
