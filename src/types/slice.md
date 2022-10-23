# Slice types
# 切片类型

>[slice.md](https://github.com/rust-lang/reference/blob/master/src/types/slice.md)\
>commit: 5ae4a38e36135e02e01933ba81d206b1c2e9ce70 \
>本章译文最后维护日期：2021-07-31

> **<sup>句法</sup>**\
> _SliceType_ :\
> &nbsp;&nbsp; `[` [_Type_] `]`

切片是一种[动态内存宽度类型(dynamically sized type)][dynamically sized type]，它代表类型为 `T` 的元素组成的数据序列的一个“视图(view)”。切片类型写为 `[T]`。

切片类型通常都是通过指针类型来使用，例如：

* `&[T]`，共享切片('shared slice')，常被直接称为切片(`slice`)。它不拥有它指向的数据，只是借用。
* `&mut [T]`，可变切片('mutable slice')。它可变借用它指向的数据。
* `Box<[T]>`, 装箱的切片('boxed slice')。

示例：

```rust
// 一个堆分配的数组，被自动强转成切片
let boxed_array: Box<[i32]> = Box::new([1, 2, 3]);

// 数组上的（共享）切片
let slice: &[i32] = &boxed_array[..];
```

切片的所有元素总是初始化过的，使用 Rust 中的安全(safe)方法或操作符来访问切片时总是会做越界检查。

[_Type_]: ../types.md#type-expressions
[dynamically sized type]: ../dynamically-sized-types.md