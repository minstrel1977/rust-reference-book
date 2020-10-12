## Behavior considered undefined
## 被认为是未定义的行为

>[behavior-considered-undefined.md](https://github.com/rust-lang/reference/blob/master/src/behavior-considered-undefined.md)\
>commit 8aa6f0f5471a23621f52d16e823c6316fda2b904

如果 Rust 代码里出现了下列列表中的任何行为，则它是不正确的。这包括非安全(`unsafe`)块和非安全(`unsafe`)函数里的代码。非安全(`unsafe`)只意味着避免未定义行为(undefined behavior)的责任在程序员；它没有改变任何关于 Rust 程序必须不能导致未定义行为的事实。

在编写非安全(`unsafe`)代码时，确保任何与非安全(`unsafe`)代码交互的安全代码不会触发下述未定义行为是程序员的责任。对于任何安全客户端，满足此属性的非安全(`unsafe`)代码称为*健壮的(sound)*；如果非安全(`unsafe`)代码可以被安全代码滥用以致出现未定义的行为，那么此非安全(`unsafe`)代码是*不健壮的(unsound)*。

<div class="warning">

***警告：*** 下面的列表并非详尽无遗。对于非安全代码中什么是允许的，什么是不允许的，Rust 还没有正式的语义模型，因此可能有更多的行为被认为是不安全的。下面的列表是仅仅是我们确定知晓的未定义行为。在编写非安全代码之前，请阅读 [Rustonomicon]。

</div>

* 数据竞争。
* (使用 `*` 操作符)解引用悬垂或未对齐裸指针。
* 破坏[指针别名规则][pointer aliasing rules]。`&mut T` 和 `&T` 遵循 LLVM 的作用域[无别名(noalias)][noalias]模型，除非 `&T` 内部包含 [`UnsafeCell<U>`] 类型。
* 修改不可变的数据。[常量(`const`)]项内的所有数据都是不可变的。此外，所有通过共享引用接触到的数据或不可变绑定所拥有的数据都是不可变的，除非该数据包含在 `UnsafeCell<U>`] 中。
* 通过编译器内部函数调用未定义行为。
* 执行用当前平台不支持的平台特性编译的代码(参见[`target_feature`])。
* 用错误的 ABI 调用函数，或从具有错误展开ABI的函数里开启展开操作。
* 产生无效值，即使在私有字段和本地变量中也是如此。“产生”值发生在任何赋值阶段或从位置表达式里读取值、传递给函数/原语操作(primitive operation)或从函数/原语操作返回值的任何时候。
  以下值无效（对应其各自的类型）：
  * 布尔型中除 `false` (`0`) 或 `true` (`1`) 之外的值。
  * 不包括在该枚举(`enum`)类型定义中的判别值。
  * 指向为空(null)的函数指针(`fn` pointer)。
  * 代理项(Surrogate)或码点大于 `char::MAX` 的字符(`char`)值。
  * `!`类型值 (此类型的所有值都是无效的).
  * 从[未初始化的内存][undef]中，或从字符串切片(`str`)的未初始化部分获取的整数(i*/u*)、浮点值(f*)或裸指针。
  * 悬垂、未对齐或指向无效值的引用或 `Box<T>`。
  * 宽(wide)引用、`Box<T>` 或原始指针中的无效元数据：
    * 如果 `dyn Trait` 元数据不是指向 `Trait` 的虚函数表(vtable)（该虚函数表与该指针或引用所指向的实际动态 trait 相匹配）的指针，则 `dyn Trait` 元数据无效。
    * 如果切片的长度不是有效的 `usize`，则该切片元数据是无效的(也就是说，不能从未初始化的内存中读取它)。
  * 带有无效值的自定义类型的值无效。在标准库中，这影响到了 [`NonNull<T>`] 和 [`NonZero*`]。

    > **注意**：`rustc` 通过未稳定的 `rustc_layout_scalar_valid_range_*` 属性实现了这一点。

**注意：** 未初始化的内存对于任何具有有限有效值集的类型也隐式无效。换句话说，允许读取未初始化内存的情况只发生在联合体(`union`)内部和“对齐填充区(padding)”里（类型的字段/元素之间的间隙）。

如果一个引用/指针是空的，或者它所指向的字节不是同一个分配的一部分(所以特别地，它们都必须是某个分配的一部分)，那么这个引用/指针就是悬空的。它所指向的字节范围由指针值和pointee类型的大小(使用val的大小)决定。因此，如果span为空，那么“悬空”与“非空”相同。注意，片和字符串指向它们的整个范围，因此长度元数据永远不要太大，这一点很重要。特别是，分配，因此切片和字符串不能大于isize::MAX字节
如果引用/指针为空或者它指向的所有字节不是同一次分配的一部分（因此，它们都必须是某个分配的一部分），那么它就是“悬垂”的。它指向的字节范围由指针值和指针对象类型的大小决定（使用 `size_of_val`）。因此，如果这个字节范围为空，“悬垂”与“非空”相同。请注意，切片和字符串指向它们的整个范围，因此切片的长度元数据永远不要太大这点很重要。尤其是，分配，因此切片和字符串不能大于isize:：MAX bytes。
A reference/pointer is "dangling" if it is null or not all of the bytes it points to are part of the same allocation (so in particular they all have to be part of *some* allocation). The span of bytes it points to is determined by the pointer value and the size of the pointee type (using `size_of_val`). As a consequence, if the span is empty, "dangling" is the same as "non-null". Note that slices and strings point to their entire range, so it is important that the length metadata is never too large. In particular, allocations and therefore slices and strings cannot be bigger than `isize::MAX` bytes.

> **Note**: Undefined behavior affects the entire program. For example, calling
> a function in C that exhibits undefined behavior of C means your entire
> program contains undefined behaviour that can also affect the Rust code. And
> vice versa, undefined behavior in Rust can cause adverse affects on code
> executed by any FFI calls to other languages.

[`const`]: items/constant-items.html
[noalias]: http://llvm.org/docs/LangRef.html#noalias
[pointer aliasing rules]: http://llvm.org/docs/LangRef.html#pointer-aliasing-rules
[undef]: http://llvm.org/docs/LangRef.html#undefined-values
[`target_feature`]: attributes/codegen.md#target_feature属性
[`UnsafeCell<U>`]: ../std/cell/struct.UnsafeCell.html
[Rustonomicon]: ../nomicon/index.html
[`NonNull<T>`]: ../core/ptr/struct.NonNull.html
[`NonZero*`]: ../core/num/index.html
