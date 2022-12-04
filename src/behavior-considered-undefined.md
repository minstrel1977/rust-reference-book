## Behavior considered undefined
## 未定义的行为

>[behavior-considered-undefined.md](https://github.com/rust-lang/reference/blob/master/src/behavior-considered-undefined.md)\
>commit: 2af818d235b6c39cb76a099b859a84438e3409f2 \
>本章译文最后维护日期：2022-12-04

如果 Rust 代码出现了下面列表中的任何行为，则此代码被认为不正确。这包括非安全(`unsafe`)块和非安全函数里的代码。非安全只意味着避免出现未定义行为(undefined behavior)的责任在程序员；它没有改变任何关于 Rust 程序必须确保不能写出导致未定义行为的代码的事实。

在编写非安全代码时，确保任何与非安全代码交互的安全代码不会触发下述未定义行为是程序员的责任。对于任何使用非安全代码的安全客户端(safe client)，如果当前条件满足了此非安全代码对于安全条件的要求，那此此非安全代码对于此安全客户端就是*健壮的(sound)*；如果非安全(`unsafe`)代码可以被安全代码滥用以致出现未定义行为，那么此非安全(`unsafe`)代码对这些安全代码来说就是*不健壮的(unsound)*。

<div class="warning">

***警告：*** 下面的列表并非详尽无遗地罗列了 Rust 中的未定义行为。而且对于在非安全代码中什么是明确不允许的，目前 Rust 还没有正式的语义模型，因此将来可能会有更多的行为被认为是不安全的。下面的列表仅仅是我们当前确定知晓的未定义行为。在编写非安全代码之前，请阅读 [Rustonomicon]。

</div>

* 数据竞争。
* 在[悬垂][dangling]或未对齐的裸指针上执行[解引用操作][dereference expression] (`*expr`)，甚至[位置表达式][place expression context](e.g. `addr_of!(&*expr)`)上也不安全。
* 破坏[指针别名规则][pointer aliasing rules]。`Box<T>`、`&mut T` 和 `&T` 遵循 LLVM 的作用域[无别名(noalias)][noalias]模型(scoped noalias model)，除非 `&T` 包含一个 [`UnsafeCell<U>`] 类型。活动的引用和 box类型的智能指针不可为悬垂[dangling]状态。活动持续时间并未明确指定，但存在一些限制条件：
  * 对于引用来说，活动持续时间由借用检查器指定的句法生存期上限来限制；它不能存活得比那个生存期*更长*。
  * 每次将引用或 box类型的智能指针传递给函数或从函数返回时，它都被视为是活动的。
  * 当引用（注意不是 box类型的智能指针！）传递给函数时，它存活的至少与该函数调用一样长，这次同样排除了 `&T` 包含了 [`UnsafeCell<U>`] 的情况。

  当这些类型的值（`Box<T>`、`&mut T` 和 `&T` 类型的值）被传递给复合类型的（内嵌）成员字段时，所有的这些规则都适用，但注意传递给间接寻址的指针不适用。
* 修改不可变的数据。[常量(`const`)][`const`]项内的所有数据都是不可变的。此外，所有通过共享引用接触到的数据或不可变绑定所拥有的数据都是不可变的，除非该数据包含在 [`UnsafeCell<U>`] 中。
* 通过编译器内部函数(compiler intrinsics)调用未定义行为。[^译注1]
* 执行基于当前平台不支持的平台特性编译的代码（参见 [`target_feature`]），*除非*此平台特别申明执行带有此特性的代码安全。
* 用错误的 ABI约定来调用函数，或使用错误的 ABI展开约定来从某函数里发起展开(unwinding)。  
* 产生非法值(invalid value)，即使在私有字段和本地变量中也是如此。“产生”值发生在这些时候：把值赋给位置表达式、从位置表达式里读取值、传递值给函数/基本运算(primitive operation)或从函数/基本运算中返回值。
  以下值非法值（相对于它们各自的类型来说）：
  * 布尔型[`bool`]中除 `false` (`0`) 或 `true` (`1`) 之外的值。
  * 不包括在该枚举(`enum`)类型定义中的判别值。
  * 指向为空(null)的函数指针(`fn` pointer)。
  * 代理码点(Surrogate)或码点大于 `char::MAX` 的字符(`char`)值。
  * `!`类型的值（任何此类型的值都是非法的）。
  * 从[未初始化的内存][undef]中，或从字符串切片(`str`)的未初始化部分获取的整数（i*/u*）、浮点值（f*）或裸指针。
  * 引用或 `Box<T>` （代表的指针）指向了[悬垂(dangling)][dangling]、未对齐或指向非法值。
  * 宽(wide)引用、`Box<T>` 或原始指针中带有非法元数据(metadata)：
    * 如果 trait对象(`dyn Trait`)的元数据不是指向 `Trait` 的虚函数表(vtable)（该虚函数表与该指针或引用所指向的实际动态 trait 相匹配）的指针，则 `dyn Trait` 元数据非法。
    * 如果切片的长度不是有效的 `usize`，则该切片的元数据是非法的（也就是说，不能从它未初始化的内存中读取它）。
  * 带有非法值的自定义类型的值非法。在标准库中，这条促成了 [`NonNull<T>`] 和 [`NonZero*`] 的出现。

    > **注意**：`rustc` 是使用还未稳定下来的属性 `rustc_layout_scalar_valid_range_*` 来验证这条规则的。
* 错误的使用内联汇编，具体细节，参见使用内联汇编编写代码时的相关[规则][rules]。

**注意：** 未初始化的内存对于任何具有有限有效值集的类型来说也隐式非法。也就是说，允许读取未初始化内存的情况只发生在联合体(`union`)内部和“对齐填充区(padding)”里（类型的字段/元素之间的间隙）。

> **注意**：未定义行为影响整个程序。例如，在 C 中调用一个 C函数已经出现了未定义行为，这意味着包含此调用的整个程序都包含了未定义行为。如果 Rust 再通过 FFI 来调用这段 C程序/代码，那这段 Rust 代码也包含了未定义行为。反之亦然。因此 Rust 中的未定义行为会对任何其他通过 FFI 过来调用的代码造成不利影响。

### Dangling pointers
### 悬垂指针

如果引用/指针为空或者它指向的所有字节不是同一次内存分配(live allocation)的一部分（因此，它们都必须是*某些*内存分配的一部分），那么它就是“悬垂”的。它指向的字节跨度(span)由指针本身和指针所指对象的类型的内存宽度决定（此内存宽度可使用 `size_of_val` 检测）。

如果类型的内存宽度为0，则该指针必定 要么指向某个初始化内存的内部（包括刚好指向分配的最后一个字节之后），要么直接从非零整型字面量来构造而来。

请注意，动态内存宽度类型（如切片和字符串）指向其底层的整个数据范围，因此他们的代表长度的元数据永远不要太大，这一点很重要。特别需要注意的是 Rust里，值的动态内存宽度（由`size_of_val` 来确定）不能超过 `isize::MAX`。

[dangling]: #dangling-pointers
[`bool`]: types/boolean.md
[`const`]: items/constant-items.md
[noalias]: http://llvm.org/docs/LangRef.html#noalias
[pointer aliasing rules]: http://llvm.org/docs/LangRef.html#pointer-aliasing-rules
[undef]: http://llvm.org/docs/LangRef.html#undefined-values
[`target_feature`]: attributes/codegen.md#the-target_feature-attribute
[`UnsafeCell<U>`]: https://doc.rust-lang.org/std/cell/struct.UnsafeCell.html
[Rustonomicon]: https://doc.rust-lang.org/nomicon/index.html
[`NonNull<T>`]: https://doc.rust-lang.org/core/ptr/struct.NonNull.html
[`NonZero*`]: https://doc.rust-lang.org/core/num/index.html
[dereference expression]: expressions/operator-expr.md#the-dereference-operator
[place expression context]: expressions.md#place-expressions-and-value-expressions
[rules]: inline-assembly.md#rules-for-inline-assembly