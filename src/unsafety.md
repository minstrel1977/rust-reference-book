# Unsafety
# 非安全性

>[unsafety.md](https://github.com/rust-lang/reference/blob/master/src/unsafety.md)\
>commit:  b0e0ad6490d6517c19546b1023948986578fc378

非安全操作是那些可能违反 Rust 静态语义里和内存安全相关的保证机制的操作。

以下语言级别的特性不能在 Rust 的 safe 子集中使用:

- 解引用[裸指针][raw pointer].
- 读取或写入[可变][mutable]或[外部][external]变量。
- 访问[联合体(`union`)]的字段，而不是赋值给它。
- 调用一个非安全(unsafe)函数(包括内部函数和外部函数)。
- 实现[非安全(unsafe) trait][unsafe trait].

[`union`]: items/unions.md
[mutable]: items/static-items.md#可变静态项
[external]: items/external-blocks.md
[raw pointer]: types/pointer.md
[unsafe trait]: items/traits.md#unsafe-traits
