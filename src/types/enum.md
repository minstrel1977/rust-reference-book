# Enumerated types
# 枚举类型

>[enum.md](https://github.com/rust-lang/reference/blob/master/src/types/enum.md)\
>commit: b0e0ad6490d6517c19546b1023948986578fc378 \
>本章译文最后维护日期：2020-11-14

*枚举类型*是一种标称型(nominal)的、异构的、不相交的类型组成的联合体类型，直接用[枚举(`enum`)数据项][`enum` item]的名称来表示。[^enumtype]

[枚举(`enum`)数据项][`enum` item]同时声明了类型和它的各种*变体(variants)*，其中每个变体都独立命名，可使用定义结构体、元组结构体或单元结构体(unit-like struct)的句法来定义它们。

枚举(`enum`)的实例可以在[枚举变体表达式][enumeration variant expression]中构造。

任何枚举值消耗的内存和其同类型的其他变体都是相同的，具体都为其枚举(`enum`)类型的最大变体所需的内存再加上存储其判别值(discriminant)所需的内存。

枚举类型不能在*结构上*表示为类型，必须通过对[枚举(`enum`)数据项][`enum` item]的命名引用(named reference)来表示。[^译注1]

[^enumtype]: ../`enum`类型类似于 ML 中的数据(`data`)构造函数声明，或 Limbo 中的 *pick ADT*。

[^译注1]: 译者理解这句话的意思是：枚举不同于普通结构化的类型，所有的枚举都是对模具数据项的引用；这里引用分两种，一种是类C枚举，就是对数据项的直接引用；另一种是带字段的枚举枚举变体，这种其实是类似于 `Box`、`Rc` 这样的命名引用类型，它通过封装其他类型来指导数据的存储和限定其上可用的操作。

[`enum` item]: ../items/enumerations.md
[enumeration variant expression]: ../expressions/enum-variant-expr.md

<!-- 2020-11-12-->
<!-- checked -->
