# Pointer types
# 指针类型

>[pointer.md](https://github.com/rust-lang/reference/blob/master/src/types/pointer.md)\
>commit: 79fcc6e4453919977b8b3bdf5aee71146c89217d \
>本译文最后维护日期：2020-10-29

Rust 中所有的指针都是显式的头等(first-class)值。它们可以被移动或复制，存储到数据结构中，或从函数中返回。

## References (`&` and `&mut`)
## 引用(`&` 和 `&mut`)

> **<sup>句法</sup>**\
> _ReferenceType_ :\
> &nbsp;&nbsp; `&` [_Lifetime_]<sup>?</sup> `mut`<sup>?</sup> [_TypeNoBounds_]

### Shared references (`&`)
### 共享引用(`&`)

共享引用(`&`)指向*由其他值拥有的*内存。创建了对值的共享引用可以防止对该值的直接更改。[内部可变性][Interior mutability]提供了在某些特定情况下的一种例外。顾名思义，对一个值的共享引用的次数没有限制。当需要指定显式的生存期时，共享引用类型被写为 `&type`，或者 `&'a type`。复制引用是一个“浅(shallow)”操作：它只涉及复制指针本身，也就是说，指针实现了 `Copy` trait。释放引用对共享引用所指向的值没有影响，但是引用[临时值][temporary value]将使此临时值在引用本身的作用域内保持存活状态。

### Mutable references (`&mut`)
### 可变引用(`&mut`)

可变引用(`&mut`)也指向其他值所拥有的内存。可变引用类型被写为 `&mut type` 或 `&'a mut type`。可变引用（其还未被借出）是访问它所指向的值的唯一方法，当然此值没有实现 `Copy` trait。

## Raw pointers (`*const` and `*mut`)
## 裸指针(`*const` 和 `*mut`)

> **<sup>句法</sup>**\
> _RawPointerType_ :\
> &nbsp;&nbsp; `*` ( `mut` | `const` ) [_TypeNoBounds_]

裸指针是没有安全性或可用性(liveness)保证的指针。裸指针写为 `*const T` 或 `*mut T`，例如，`*const i32` 表示指向 32-bit 有符号整数的裸指针。复制或销毁裸指针对任何其他值的生命周期都没有影响。对裸指针的解引用是[非安全(`unsafe`)操作][`unsafe` operation]，可以通过重新借用裸指针（`&*` 或 `&mut *`）将其转换为引用。在 Rust 代码中通常不鼓励使用裸指针；它们的存在是为了支持与外部代码的互操作性，以及编写对性能要求很高的函数或低级的函数。

在比较裸指针时，比较的是它们的地址，而不是它们指向的数据。当比较裸指针和[动态尺寸类型][dynamically sized types]时，还会比较它们指针上的附加/元数据。

## Smart Pointers
## 智能指针

标准库包含了额外的“智能指针”类型，它们提供了在引用和裸指针这类低级指针之外的更多的功能。

[Interior mutability]: ../interior-mutability.md
[_Lifetime_]: ../trait-bounds.md
[_TypeNoBounds_]: ../types.md#type-expressions
[`unsafe` operation]: ../unsafety.md
[dynamically sized types]: ../dynamically-sized-types.md
[temporary value]: ../expressions.md#temporaries

<!-- 2020-11-3 -->
<!-- checked -->
