# Unsafe blocks
# 非安全块

>[unsafe-blocks.md](https://github.com/rust-lang/reference/blob/master/src/unsafe-blocks.md)\
>commit  4a2bdf896cd2df370a91d14cb8ba04e326cd21db

一个代码块可以以关键字 `unsafe` 作为前缀，以允许在安全(safe)函数中调用非安全(`unsafe`)函数或对裸指针做解引用操作。

当程序员确信一系列潜在的非安全操作实际上是安全的，他们可以将这段代码序列(作为一个整体)封装在一个非安全(`unsafe`)块中。编译器将认为在当前的上下文中使用这样的代码是安全的。

非安全块用于封装外部库、直接操作硬件或实现语言中没有直接提供的特性。例如，Rust 在该语言中提供了实现内存安全并发所需的语言特性，但是线程和消息传递的实现是在标准库中实现的。
Unsafe blocks are used to wrap foreign libraries, make direct use of hardware or implement features not directly present in the language. For example, Rust provides the language features necessary to implement memory-safe concurrency in the language but the implementation of threads and message passing is in the standard library.

Rust的类型系统是动态安全需求的保守近似值，因此在某些情况下使用安全代码会带来性能损失。例如，双向链表不是树结构，只能在安全代码中使用引用计数指针表示。通过使用非安全(`unsafe`)块将反向链接表示为原始指针，它可以只用方框来实现。
Rust's type system is a conservative approximation of the dynamic safety requirements, so in some cases there is a performance cost to using safe code. For example, a doubly-linked list is not a tree structure and can only be represented with reference-counted pointers in safe code. By using `unsafe` blocks to represent the reverse links as raw pointers, it can be implemented with only boxes.
