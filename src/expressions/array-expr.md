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

也可以方括号内放置两用个分号(`;`)分隔的表达式。这种形式里，分号(`;`)后面的表达式必须具有 `usize` 类型，并且必须是[常量表达式][constant expression]，例如[字面量](../tokens.md#字面量)或[常量项](../items/constant-items.md)。`[a; b]` 形式创建的数组，语义为数组内包含 `b` 个 `a` 值的副本。如果分号后面的表达式的值大于1，则要求 `a` 的类型实现 [`Copy`](../special-types-and-traits.md#copy)。

```rust
[1, 2, 3, 4];
["a", "b", "c", "d"];
[0; 128];              // 内含128个0的数组
[0u8, 0u8, 0u8, 0u8,];
[[1, 0, 0], [0, 1, 0], [0, 0, 1]]; // 二维数组
```

### Array expression attributes
### 数组表达式上的属性

[Inner attributes] are allowed directly after the opening bracket of an array
expression in the same expression contexts as [attributes on block
expressions].

## Array and slice indexing expressions
## 数组和切片索引表达式

> **<sup>句法</sup>**\
> _IndexExpression_ :\
> &nbsp;&nbsp; [_Expression_] `[` [_Expression_] `]`

[Array](../types/array.md) and [slice](../types/slice.md)-typed expressions can be
indexed by writing a square-bracket-enclosed expression of type `usize` (the
index) after them. When the array is mutable, the resulting [memory location]
can be assigned to.

For other types an index expression `a[b]` is equivalent to
`*std::ops::Index::index(&a, b)`, or
`*std::ops::IndexMut::index_mut(&mut a, b)` in a mutable place expression
context. Just as with methods, Rust will also insert dereference operations on
`a` repeatedly to find an implementation.

Indices are zero-based for arrays and slices. Array access is a [constant
expression], so bounds can be checked at compile-time with a constant index
value. Otherwise a check will be performed at run-time that will put the thread
in a _panicked state_ if it fails.

```rust,should_panic
// lint is deny by default.
#![warn(unconditional_panic)]

([1, 2, 3, 4])[2];        // Evaluates to 3

let b = [[1, 0, 0], [0, 1, 0], [0, 0, 1]];
b[1][2];                  // multidimensional array indexing

let x = (["a", "b"])[10]; // warning: index out of bounds

let n = 10;
let y = (["a", "b"])[n];  // panics

let arr = ["a", "b"];
arr[10];                  // warning: index out of bounds
```

The array index expression can be implemented for types other than arrays and slices
by implementing the [Index] and [IndexMut] traits.

[IndexMut]: https://doc.rust-lang.org/std/ops/trait.IndexMut.html
[Index]: https://doc.rust-lang.org/std/ops/trait.Index.html
[Inner attributes]: ../attributes.md
[_Expression_]: ../expressions.md
[_InnerAttribute_]: ../attributes.md
[attributes on block expressions]: block-expr.md#块表达式上的属性
[constant expression]: ../const_eval.md#常量表达式
[memory location]: ../expressions.md#位置表达式和值表达式