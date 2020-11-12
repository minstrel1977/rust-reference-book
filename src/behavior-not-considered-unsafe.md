## Behavior not considered `unsafe`
## 不被认为是非安全的行为

>[behavior-not-considered-unsafe.md](https://github.com/rust-lang/reference/blob/master/src/behavior-not-considered-unsafe.md)\
>commit: 75fd23737cd08ec1ae14deae5e2680d9007575ae \
>本章译文最后维护日期：2020-11-2

虽然程序员可能（应该）发现下列行为是不良的、意外的或错误的，但 Rust 编译器并不认为这些行为是*非安全的(unsafe)*。

##### 死锁(Deadlocks)
##### 内存和其他资源的泄漏(Leaks of memory and other resources)
##### 退出而不调用析构函数(Exiting without calling destructors)
##### 通过指针泄漏暴露随机基地址(Exposing randomized base addresses through pointer leaks)
##### 整数溢出(Integer overflow)

如果程序包含算术溢出(arithmetic overflow)，则说明程序员犯了错误。在下面的讨论中，我们将区分算术溢出和包装算法(wrapping arithmetic)。前者是错误的，而后者是有意为之的。

当程序员启用了 `debug_assert!` 断言（例如，通过启用非优化的构建方式），相应的实现就必须插进来以便在溢出时触发 `panic`。而其他类型的构建形式则有可能也在溢出时触发 `panics`，或仅仅隐式包装一下溢出值，对溢出过程做静音处理。也就是说具体怎么对待溢出由插进来的编译实现决定。

在隐式包装溢出的情况下，（编译器实现的包装算法）实现必须通过使用2的补码溢出(two's complement overflow)约定来提供定义良好的（即使仍然被认为是错误的）溢出包装结果。

整型提供了一些固有方法，允许程序员显式地执行包装算法。例如，`i32::wrapping_add` 提供了使用2的补码溢出约定算法的加法，即包装类加法(wrapping addition)。

标准库还提供了一个 `Wrapping<T>` 的新类型，该类型确保 `T` 的所有标准算术操作都具有包装语义。

请参阅 [RFC 560] 以了解错误条件、基本原理以及有关整数溢出的更多详细信息。

[RFC 560]: https://github.com/rust-lang/rfcs/blob/master/text/0560-integer-overflow.md

<!-- 2020-11-12-->
<!-- checked -->
