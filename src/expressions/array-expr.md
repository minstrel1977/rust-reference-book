# Array and array index expressions
# 数组和数组索引表达式

>[array-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/array-expr.md)\
>commit 65c523479abb8024672918444ff839426ff5c3a7

## Array expressions
## 数组表达式

> **<sup>句法</sup>**\
> _ArrayExpression_ :\
> &nbsp;&nbsp; `[` [_InnerAttribute_]<sup>\*</sup> _ArrayElements_<sup>?</sup> `]`
>
> _ArrayElements_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_Expression_] ( `,` [_Expression_] )<sup>\*</sup> `,`<sup>?</sup>\
> &nbsp;&nbsp; | [_Expression_] `;` [_Expression_]

[数组](../types/array.md)表达式可以通过在方括号中括起零个或多个统一类型的逗号分隔的表达式来编写。这样编写将生成一个包含这些值的写入顺序的数组。

也可以方括号内放置两用个分号(`;`)分隔的表达式。这种形式里，分号(`;`)后面的表达式必须具有 `usize` 类型，并且必须是[常量表达式][constant expression]，例如[字面量](../tokens.md#字面量)或[常量项](../items/constant-items.md)。`[a; b]` 形式创建的数组，语义为数组内包含 `b` 个 `a` 值的副本。如果分号后面的表达式的值大于1，则要求 `a` 的类型实现了 [`Copy`](../special-types-and-traits.md#copy)。

```rust
[1, 2, 3, 4];
["a", "b", "c", "d"];
[0; 128];              // 内含128个0的数组
[0u8, 0u8, 0u8, 0u8,];
[[1, 0, 0], [0, 1, 0], [0, 0, 1]]; // 二维数组
```

### Array expression attributes
### 数组表达式上的属性

适用于[块表达式上的属性][attributes on block expressions]的表达式上下文同样适用于数组表达式上的属性，同样也是允许[内部属性][Inner attributes]直接位于表达式的左括号之后。

## Array and slice indexing expressions
## 数组和切片索引表达式

> **<sup>句法</sup>**\
> _IndexExpression_ :\
> &nbsp;&nbsp; [_Expression_] `[` [_Expression_] `]`

[数组](../types/array.md)和[切片](../types/slice.md)类型表达式可以通过在它们后面编写一个类型为 `usize` 的方括号括起来的表达式(索引)来进行索引检索。如果数组是可变的，则其相应的[内存位置][memory location]还可以被赋值。

对于非数组/切片类型的索引表达式 `a[b]` 相当于 `*std::ops::Index::index(&a, b)` 或者在可变位置表达式上下文中相当于 `*std::ops::IndexMut::index_mut(&mut a, b)`。与普通方法一样，Rust也将在 `a` 上反复插入解引用操作，以查找到对上述方法的实现。

数组和切片的索引是从零开始的。数组访问是一个[常量表达式][constant expression]，因此可以在编译时使用常量索引值检查边界。否则，将在运行时执行检查，如果边界检查未通过，那将使当前线程处于 *panick状态*。

```rust,should_panic
// 默认情况下,`unconditional_panic` lint检查会执行执行deny级别的设置，
// 即crate在默认情况下会有外部属性设置`#[deny(unconditional_panic)]`
// 而`(["a", "b"])[n]`里简单的动态索引值会被该lint检查出来，而提前报错，导致程序被拒绝编译。
// 因此这里调低`unconditional_panic`的lint级别以通过编译。
#![warn(unconditional_panic)]

([1, 2, 3, 4])[2];        // 3

let b = [[1, 0, 0], [0, 1, 0], [0, 0, 1]];
b[1][2];                  // 多维数组索引

let x = (["a", "b"])[10]; // 告警：下标越界

let n = 10; 
// 译者注：上行可以在`#![warn(unconditional_panic)]`被注释的情况下换成
// `let n = if true {10} else {0};` 
// 试试,这行就不会被unconditional_panic lint 检查到了
  
let y = (["a", "b"])[n];  // panic

let arr = ["a", "b"];
arr[10];                  // 告警：下标越界
```

数组索引表达式可以通过实现[Index]和[IndexMut] trait来为数组和切片片以外的类型实现。

[IndexMut]: https://doc.rust-lang.org/std/ops/trait.IndexMut.html
[Index]: https://doc.rust-lang.org/std/ops/trait.Index.html
[Inner attributes]: ../attributes.md
[_Expression_]: ../expressions.md
[_InnerAttribute_]: ../attributes.md
[attributes on block expressions]: block-expr.md#块表达式上的属性
[constant expression]: ../const_eval.md#常量表达式
[memory location]: ../expressions.md#位置表达式和值表达式