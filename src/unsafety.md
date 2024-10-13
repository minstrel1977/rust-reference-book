# Unsafety
# 非安全性

>[unsafety.md](https://github.com/rust-lang/reference/blob/master/src/unsafety.md)\
>commit:  5bbf37952fe1673dd9134880d21ff48ad04c54e1 \
>本章译文最后维护日期：2024-10-13

非安全操作(Unsafe operations)是那些可能潜在地违反 Rust 静态语义里的和内存安全保障相关的操作。

以下语言级别的特性不能在 Rust 的安全(safe)子集中使用:

- 读取或写入[可变][mutable]静态变量；读取或写入或[外部][external]静态变量。
- 访问[联合体(`union`)]的字段，注意不是给它的字段赋值。
- 调用一个非安全(unsafe)函数（包括外部函数和和内部函数(intrinsic)）。
- 实现[非安全(unsafe) trait][unsafe trait].
- 声明一个 [`extern`]块.
- 为程序项应用[非安全属性][unsafe attribute]。

[`extern`]: items/external-blocks.md
[`union`]: items/unions.md
[mutable]: items/static-items.md#mutable-statics
[external]: items/external-blocks.md
[raw pointer]: types/pointer.md
[unsafe trait]: items/traits.md#unsafe-traits
[unsafe attribute]: attributes.md
