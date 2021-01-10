# Tuple types
# 元组类型

>[tuple.md](https://github.com/rust-lang/reference/blob/master/src/types/tuple.md)\
>commit: 97ed23e824990cef2db1e95521baf003e1bf3bcd \
>本章译文最后维护日期：2021-1-10

> **<sup>句法</sup>**\
> _TupleType_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `(` `)`\
> &nbsp;&nbsp; | `(` ( [_Type_] `,` )<sup>+</sup> [_Type_]<sup>?</sup> `)`
> 
*元组类型*是把不同类型放在同一个列表中异构组成结构类型[^1]。这个列表中的每个条目(entry)都是对应元组的一个*元素*[^2]。元素可以用它在列表中的位置数字来索引，读作*第 n 个元素*，索引从 0(`0`)开始。

元素的数量决定元组的元数(arity)。有 `n` 个元素的元组叫做 n元元组(`n-ary tuple`)。例如，有两个元素的元组就是二元元组。

出于方便和历史原因，不带元素(`()`)的元组类型通常被称为*单元(unit)*或*单元类型(unit type)*。它的值也被称为*单元*或*单元值*。

元组类型是通过在圆括号封闭的逗号分隔的列表中列出其元素的类型来编写的。一元元组的元素类型后面需要一个逗号，以便和[圆括号组合类型(parenthesized type)][parenthesized type]区分开来。

元组类型的示例：

* `()` (单元)
* `(f64, f64)`
* `(String, i32)`
* `(i32, String)` (跟前一个示例类型不一样)
* `(i32, f64, Vec<String>, Option<bool>)`

这种类型的值是使用[元组表达式][tuple expression]来构造的。此外，如果没有其他有意义的值可供求得/返回，很多种表达式都将生成单元值。元组元素可以通过[元组索引表达式][tuple index expression]或[模式匹配][pattern matching]来访问。

[^1]: 如果一些类型相互比较，发现它们对等位置的内部类型是相等的，那么这些结构化类型就总是相等的。有关元组的标称类型版本，请参见[元组结构体][tuple structs]。

[^2]: 元素与字段等效，只是元素用数字索引代替标识符来标识和检索。

[_Type_]: ../types.md#type-expressions
[parenthesized type]: ../types.md#parenthesized-types
[pattern matching]: ../patterns.md#tuple-patterns
[tuple expression]: ../expressions/tuple-expr.md#tuple-expressions
[tuple index expression]: ../expressions/tuple-expr.md#tuple-indexing-expressions
[tuple structs]: ./struct.md

<!-- 2021-1-10-->
<!-- checked -->
