# Pointer types
# 指针类型

>[pointer.md](https://github.com/rust-lang/reference/blob/master/src/types/pointer.md)\
>commit 79fcc6e4453919977b8b3bdf5aee71146c89217d

Rust 中的所有指针都是显式的头等(first-class)值。它们可以被移动或复制，存储到数据结构中，或从函数中返回。

## References (`&` and `&mut`)
## 引用(`&` 和 `&mut`)

> **<sup>句法</sup>**\
> _ReferenceType_ :\
> &nbsp;&nbsp; `&` [_Lifetime_]<sup>?</sup> `mut`<sup>?</sup> [_TypeNoBounds_]

### Shared references (`&`)
### 共享引用(`&`)

共享引用(`&`)指向*由其他值拥有的*内存。创建对值的共享引用时，可以防止对值的直接更改。在某些情况下，[内部可变性][Interior mutability]提供了一个例外。顾名思义，对一个值的共享引用的次数没有限制。当需要指定显式的生命周期时，共享引用类型被写为 `&type`，或者 `&'a type`。复制引用是一个“浅(shallow)”操作：它只涉及复制指针本身，也就是说，指针拥有 `Copy` 特性。释放引用对共享引用所指向的值没有影响，但是引用[临时值][temporary value]将使此临时值在引用本身的作用域内保持可用状态。

### Mutable references (`&mut`)
### 可变引用(`&mut`)

可变引用(`&mut`)也指向其他值所拥有的内存。可变引用类型被写为 `&mut type` 或 `&'a mut type`。可变引用（还没有被借用）是访问它所指向的值的唯一方法，当然此值没有实现 `Copy` trait。

## Raw pointers (`*const` and `*mut`)
## 裸指针(`*const` 和 `*mut`)

> **<sup>句法</sup>**\
> _RawPointerType_ :\
> &nbsp;&nbsp; `*` ( `mut` | `const` ) [_TypeNoBounds_]

裸指针是没有安全性或可用性(liveness)保证的指针。裸指针写为 `*const T` 或 `*mut T`，例如，`*const i32` 表示指向有符号32位整数的裸指针。复制或销毁裸指针不会影响任何其他值的生命周期。取消对裸指针的引用是不安全的操作，也可以通过重新引导裸指针（&amp;*或&amp;mut*）将其转换为引用。在Rust代码中通常不鼓励使用裸指针；它们的存在是为了支持与外部代码的互操作性，以及编写性能关键的或低级的函数。
Raw pointers are pointers without safety or liveness guarantees. Raw pointers
are written as `*const T` or `*mut T`, for example `*const i32` means a raw
pointer to a 32-bit integer. Copying or dropping a raw pointer has no effect
on the lifecycle of any other value. Dereferencing a raw pointer is an
[`unsafe` operation], this can also be used to convert a raw pointer to a
reference by reborrowing it (`&*` or `&mut *`). Raw pointers are generally
discouraged in Rust code; they exist to support interoperability with foreign
code, and writing performance-critical or low-level functions.

When comparing raw pointers they are compared by their address, rather than by
what they point to. When comparing raw pointers to [dynamically sized types] they
also have their additional data compared.

## Smart Pointers

The standard library contains additional 'smart pointer' types beyond references
and raw pointers.

[Interior mutability]: ../interior-mutability.md
[_Lifetime_]: ../trait-bounds.md
[_TypeNoBounds_]: ../types.md#type-expressions
[`unsafe` operation]: ../unsafety.md
[dynamically sized types]: ../dynamically-sized-types.md
[temporary value]: ../expressions.md#临时位置
