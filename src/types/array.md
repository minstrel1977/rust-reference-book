# Array types
# 数组类型

>[array.md](https://github.com/rust-lang/reference/blob/master/src/types/array.md)\
>commit 2f459e22ec30a94bafafe417da4e95044578df73

> **<sup>句法</sup>**\
> _ArrayType_ :\
> &nbsp;&nbsp; `[` [_Type_] `;` [_Expression_] `]`

数组是 `N`个类型`T` 的固定长度(fixed-size)的元素序列，数组类型写为 `[T; N]`。长度(size)是一个计算结果为 [`usize`] 的[常量表达式][constant expression]。

示例：

```rust
// 一个栈分配的数组
let array: [i32; 3] = [1, 2, 3];

// 一个堆分配的数组，被强转成切片
let boxed_array: Box<[i32]> = Box::new([1, 2, 3]);
```

数组的所有元素总是初始化过的，使用 Rust 的安全(safe)方法或操作符来访问数组时总是会做越界检查。

> 注意：标准库类型提供了堆分配的可调整大小的数组类型 [`Vec<T>`]。

[_Expression_]: ../expressions.md
[_Type_]: ../types.md#type-expressions
[`Vec<T>`]: https://doc.rust-lang.org/std/vec/struct.Vec.html
[`usize`]: numeric.md#machine-dependent-integer-types
[constant expression]: ../const_eval.md#常量表达式
