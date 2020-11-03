# Enumerated types
# 枚举类型

>[enum.md](https://github.com/rust-lang/reference/blob/master/src/types/enum.md)\
>commit: b0e0ad6490d6517c19546b1023948986578fc378 \
>本译文最后维护日期：2020-10-29

*枚举类型*是一种标称型(nominal)的、异构的(heterogeneous)、不相交的联合类型(disjoint union type)，具体的枚举类型由[枚举(`enum`)数据项][`enum` item]的名称来表示。[^enumtype]

[枚举(`enum`)数据项][`enum` item]的声明同时声明了它的类型和*变体(variants)*，其中每个变体都独立命名，可使用定义结构体、元组结构体或单元结构体(unit-like struct)的句法来定义它们。

枚举(`enum`)的实例可以在[枚举变体表达式][enumeration variant expression]中构造。

任何枚举值消耗的内存和其同类型的其他变体都是相同的，具体都为其枚举类型的最大变体所需的内存再加上存储其判别值(discriminant)所需的内存。

枚举类型不能直接表示为类型，必须通过对其[枚举(`enum`)数据项][`enum` item]的名称的引用来表示。

[^enumtype]: `enum`类型类似于 ML 中的数据(`data`)构造函数声明，或 Limbo 中的 *pick ADT*。

[`enum` item]: ../items/enumerations.md
[enumeration variant expression]: ../expressions/enum-variant-expr.md

<!-- 2020-11-3 -->
<!-- checked -->
