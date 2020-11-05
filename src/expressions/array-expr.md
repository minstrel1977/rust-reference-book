# Array and array index expressions
# 数组和数组索引表达式

>[array-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/array-expr.md)\
>commit: 65c523479abb8024672918444ff839426ff5c3a7 \
>本译文最后维护日期：2020-10-26

## Array expressions
## 数组表达式

> **<sup>句法</sup>**\
> _ArrayExpression_ :\
> &nbsp;&nbsp; `[` [_InnerAttribute_]<sup>\*</sup> _ArrayElements_<sup>?</sup> `]`
>
> _ArrayElements_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_Expression_] ( `,` [_Expression_] )<sup>\*</sup> `,`<sup>?</sup>\
> &nbsp;&nbsp; | [_Expression_] `;` [_Expression_]

[数组](https://doc.rust-lang.org/types/array.md)表达式可以通过在方括号中放置零个或多个统一类型的逗号分隔的表达式来编写。这样编写将生成一个包含这些值的数组，其中元素的顺序就是其写入的顺序。

也可以方括号内放置两用个分号(`;`)分隔的表达式。这种形式里，分号后面的表达式必须是 `usize` 类型的，并且必须是[常量表达式][constant expression]，例如[字面量](https://doc.rust-lang.org/tokens.md#literals)或[常量项](https://doc.rust-lang.org/items/constant-items.md)。`[a; b]` 形式创建的数组，语义为该数组内包含 `b` 个 `a` 值的副本。如果分号后面的表达式的值大于 1，则要求 `a` 的类型实现了 [`Copy`](https://doc.rust-lang.org/special-types-and-traits.md#copy)。

```rust
[1, 2, 3, 4];
["a", "b", "c", "d"];
[0; 128];              // 内含128个0的数组
[0u8, 0u8, 0u8, 0u8,];
[[1, 0, 0], [0, 1, 0], [0, 0, 1]]; // 二维数组
```

### Array expression attributes
### 数组表达式上的属性

允许[块表达式上的属性][attributes on block expressions]的表达式上下文同样允许数组表达式上的属性，即在这种上下文下[内部属性][Inner attributes]可直接位于表达式的左括号之后。

## Array and slice indexing expressions
## 数组和切片索引表达式

> **<sup>句法</sup>**\
> _IndexExpression_ :\
> &nbsp;&nbsp; [_Expression_] `[` [_Expression_] `]`

[数组](https://doc.rust-lang.org/types/array.md)表达式和[切片](https://doc.rust-lang.org/types/slice.md)类型的表达式可以通过后跟一个由方括号封闭一个类型为 `usize` 的表达式(索引)的方式来对此数组或切片进行索引检索。如果数组是可变的，则其检索出的[内存位置][memory location]还可以被赋值。

对于非数组/切片类型之外的索引表达式 `a[b]` 其实相当于 `*std::ops::Index::index(&a, b)`， 或者在可变位置表达式上下文中相当于 `*std::ops::IndexMut::index_mut(&mut a, b)`。与普通方法一样，Rust也将在 `a` 上反复插入解引用操作，直到查找到对上述方法的实现。

数组和切片的索引是从零开始的。数组访问是一个[常量表达式][constant expression]，因此数组索引的越界检查可以在编译时通过检查常量索引值进行。否则，越界检查将在运行时执行，如果越界检查未通过，那将把当时线程置于 *panicked 状态*。

```rust,should_panic
// 默认情况下,`unconditional_panic` lint检查会执行 deny 级别的设置，
// 即 crate 在默认情况下会有外部属性设置 `#[deny(unconditional_panic)]`
// 而像 `(["a", "b"])[n]` 这样的简单动态索引检索会被该 lint 检查出来，而提前报错，导致程序被拒绝编译。
// 因此这里调低 `unconditional_panic` 的 lint 级别以通过编译。
#![warn(unconditional_panic)]

([1, 2, 3, 4])[2];        // 3

let b = [[1, 0, 0], [0, 1, 0], [0, 0, 1]];
b[1][2];                  // 多维数组索引

let x = (["a", "b"])[10]; // 告警：索引越界

let n = 10; 
// 译者注：上行可以在 `#![warn(unconditional_panic)]` 被注释的情况下换成
// let n = if true {10} else {0};
// 试试，那下行就不会被 unconditional_panic lint 检查到了
  
let y = (["a", "b"])[n];  // panic

let arr = ["a", "b"];
arr[10];                  // 告警：索引越界
```

数组索引表达式可以被数组和切片片以外的类型通过实现 [Index] trait 和 [IndexMut] trait 来实现。

[IndexMut]: https://doc.rust-lang.org/std/ops/trait.IndexMut.html
[Index]: https://doc.rust-lang.org/std/ops/trait.Index.html
[Inner attributes]: https://doc.rust-lang.org/attributes.md
[_Expression_]: https://doc.rust-lang.org/expressions.md
[_InnerAttribute_]: https://doc.rust-lang.org/attributes.md
[attributes on block expressions]: block-expr.md#attributes-on-block-expressions
[constant expression]: https://doc.rust-lang.org/const_eval.md#constant-expressions
[memory location]: https://doc.rust-lang.org/expressions.md#place-expressions-and-value-expressions

<!-- 2020-11-3 -->
<!-- checked -->
