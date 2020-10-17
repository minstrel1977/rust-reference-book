# Slice types
# 切片类型

>[slice.md](https://github.com/rust-lang/reference/blob/master/src/types/slice.md)\
>commit: b0e0ad6490d6517c19546b1023948986578fc378

> **<sup>句法</sup>**\
> _SliceType_ :\
> &nbsp;&nbsp; `[` [_Type_] `]`

切片是一种[动态尺寸类型][dynamically sized type]，它代表内存上类型 `T` 的一系列元素的一个“视图(view)”。切片类型写为 `[T]`。

要使用切片类型，通常必须在指针后面使用，例如：

* `&[T]`，共享切片(`shared slice`)，常被直接称为切片(`slice`)，它不拥有它指向的数据，只是借用。
* `&mut [T]`，可变切片(`mutable slice`)，可变借用它指向的数据。
* `Box<[T]>`, boxed切片(`boxed slice`)。

示例：

```rust
// 一个堆分配的数组，被自动强转成切片
let boxed_array: Box<[i32]> = Box::new([1, 2, 3]);

// 数组中的（共享）切片A (shared) slice into an array
let slice: &[i32] = &boxed_array[..];
```

切片的所有元素总是被初始化过的，使用 Rust 的安全(safe)方法或操作符来访问切片时总是会做越界检查。

[_Type_]: ../types.md#type-expressions
[dynamically sized type]: ../dynamically-sized-types.md
