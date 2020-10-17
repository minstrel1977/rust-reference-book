## Behavior not considered `unsafe`
## 不被认为是非安全的行为

>[behavior-not-considered-unsafe.md](https://github.com/rust-lang/reference/blob/master/src/behavior-not-considered-unsafe.md)\
>commit: 75fd23737cd08ec1ae14deae5e2680d9007575ae

虽然程序员可能(应该)发现下列行为是不希望的、意外的或错误的，但 Rust 编译器并不认为这些行为是*非安全的*。

##### 死锁
##### 内存和其他资源的泄漏
##### 退出而不调用析构函数
##### Exposing randomized base addresses through pointer leaks
##### 整数溢出

如果程序包含算术溢出(arithmetic overflow)，则说明程序员犯了错误。在下面的讨论中，我们将区分算术溢出和包装算法(wrapping arithmetic)。前者是错误的，而后者是有意为之的。

当程序员启用了 `debug_assert!` 断言（例如，通过启用非优化的构建），实现必须插入在溢出时 `panic` 的动态检查。其他类型的构建形式也可能导致溢出时出现 `panics` 或悄无声息地包装值，具体由编译实现决定。

在隐式包装溢出的情况下，实现必须通过使用2的补码溢出(two's complement overflow)约定来提供定义良好的（即使仍然被认为是错误的）溢出包装结果。

整型提供了一些固有方法，允许程序员显式地执行包装算法。例如，`i32::wrapping_add` 提供了使用二的补码溢出约定算法的加法，即包装加法。

标准库还提供了一个 `Wrapping<T>` 的新类型，该类型确保 `T` 的所有标准算术操作都具有包装语义。

请参阅 [RFC 560] 以了解错误条件、基本原理以及有关整数溢出的更多详细信息。

[RFC 560]: https://github.com/rust-lang/rfcs/blob/master/text/0560-integer-overflow.md