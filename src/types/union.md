# Union types
# 联合体类型

>[union.md](https://github.com/rust-lang/reference/blob/master/src/types/union.md)\
>commit: b0e0ad6490d6517c19546b1023948986578fc378

*联合体类型*是一种标称型(nominal)的异构的类似C语言里的联合体的类型，具体的类型名称由[联合体(`union`)数据项][item]的名称表示。

联合体没有“活跃字段(active field)”的概念。相反，每次对联合体的访问都将联合体的部分存储内容转换为被访问字段的类型。由于转换可能会导致意外或未定义行为，所以读取联合体字段，或写入未实现 [`Copy`] 的联合体字段时都需要使用 `unsafe`。有关详细信息，请参阅[数据项][item]文档。

默认情况下，联合体(`union`)的内存布局是未定义的，但是可以使用 `#[repr(...)]`属性来提前固定住其类型布局。

[`Copy`]: ../special-types-and-traits.md#copy
[item]: ../items/unions.md
