# Union types
# 联合体类型

>[union.md](https://github.com/rust-lang/reference/blob/master/src/types/union.md)\
>commit: 969e45e09c68a167ecf456901f490cd6a21a70a2 \
>本章译文最后维护日期：2022-08-21

*联合体类型*是一种标称型(nominal)的、异构的、类似C语言里的 union 的类型，具体的类型名称由[联合体(`union`)程序项][item]的名称表示。

联合体没有“活跃字段(active field)”的概念。相反，每次对联合体的访问都会将联合体的部分存储内容转化为被访问字段的类型。
由于这种转化可能会导致意外或未定义行为，所以读取联合体字段需要用到 `unsafe`，联合体字段的类型也被限制为一些类型子集，这确保了它们永远不需要销毁。有关详细信息，请参阅[程序项][item]文档。

默认情况下，联合体(`union`)的内存布局是未定义的（特别是它的字段可以不从相对的0地址开始），但是可以使用 `#[repr(...)]`属性来固定为某一类型布局。

[`Copy`]: ../special-types-and-traits.md#copy
[`ManuallyDrop<_>`]: https://doc.rust-lang.org/std/mem/struct.ManuallyDrop.html
[item]: ../items/unions.md
