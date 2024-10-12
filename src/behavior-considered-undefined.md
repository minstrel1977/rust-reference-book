## Behavior considered undefined
## 未定义的行为

>[behavior-considered-undefined.md](https://github.com/rust-lang/reference/blob/master/src/behavior-considered-undefined.md)\
>commit: d385d421e3f450b2da4aaed4166f59598a26dd2a \
>本章译文最后维护日期：2024-10-13

如果 Rust 代码出现了下面列表中的任何行为，则此代码被认为不正确。这包括非安全(`unsafe`)块和非安全函数里的代码。非安全只意味着避免出现未定义行为(undefined behavior)的责任在程序员；它没有改变任何关于 Rust 程序必须确保不能写出导致未定义行为的代码的事实。

在编写非安全代码时，确保任何与非安全代码交互的安全代码不会触发下述未定义行为是程序员的责任。对于任何使用非安全代码的安全客户端(safe client)，如果当前条件满足了此非安全代码对于安全条件的要求，那此此非安全代码对于此安全客户端就是*健壮的(sound)*；如果非安全(`unsafe`)代码可以被安全代码滥用以致出现未定义行为，那么此非安全(`unsafe`)代码对这些安全代码来说就是*不健壮的(unsound)*。

> [!WARNING]
>  下面的列表并非详尽无遗地罗列了 Rust 中的未定义行为; 以后还有可能增添或删减。
对于在非安全代码中什么是允许的和什么是不允许的内容，目前 Rust 还没有正式的语义模型，因此可能会有更多的行为被承认为是不安全的。同时我们还保留着在将来重新定义该列表中的某些行为的权利。换言之，这个列表并没有说在未来的所有 Rust版本中，这里的任何东西都*肯定*是未定义行为（但我们也可能会在未来对这些列表项做出这样的承诺）。
>

> 在编写非安全代码之前，请阅读 [Rustonomicon]。

* 数据竞争。
* 存取基于[悬垂][dangling]或[未对齐的指针][based on a misaligned pointer]上的地址。
* 违反[界内指针算术偏移](pointer#method.offset)要求的地址映射操作。  
* 破坏[指针别名规则][pointer aliasing rules]。`Box<T>`、`&mut T` 和 `&T` 遵循 LLVM 的作用域[无别名(noalias)][noalias]模型(scoped noalias model)，除非 `&T` 包含一个 [`UnsafeCell<U>`] 类型。活动的引用和 box类型的智能指针不可为悬垂[dangling]状态。活动持续时间并未明确指定，但存在一些限制条件：
  * 对于引用来说，活动持续时间由借用检查器指定的句法生存期上限来限制；它不能存活得比那个生存期*更长*。
  * 每次将引用或 box类型的智能指针传递给函数或从函数返回时，它都被视为是活动的。
  * 当引用（注意不是 box类型的智能指针！）传递给函数时，它存活的至少与该函数调用一样长，这次同样排除了 `&T` 包含了 [`UnsafeCell<U>`] 的情况。

  当这些类型的值（`Box<T>`、`&mut T` 和 `&T` 类型的值）被传递给复合类型的（内嵌）成员字段时，所有的这些规则都适用，但注意传递给间接寻址的指针不适用。
* 修改不可变的字节数据。[常量(`const`)][`const`]项内或做隐式[常量提升][const-promoted]的表达式内的所有字节都是不可变的。
  不可变绑定或不可变`static` 所拥有的字节数据是不可变的，除非这些字节是 [`UnsafeCell<U>`] 的一部分。
  
  此外，共享引用[指向][pointed to]的字节数据是不可变的，包括通过其他引用（共享的和可变的）和 `Box`方式传递过来的；这里传递过来的（也就是传递性）包括那些存储在复合类型成员字段中的各种引用。

  修改是指与相关字节位上超过0字节的任何写入（即使该写入不会更改内存内容）。
  
* 通过编译器内部函数(compiler intrinsics)调用未定义行为。[^译注1]
* 执行基于当前平台不支持的平台特性编译的代码（参见 [`target_feature`]），*除非*此平台特别申明执行带有此特性的代码安全。
* 用错误的 ABI约定来调用函数，或使用错误的 ABI展开约定来从某函数里发起展开(unwinding)。  
* 产生[非法值][invalid-values]，即使在私有字段和本地变量中也是如此。“产生”值发生在这些时候：把值赋给位置表达式、从位置表达式里读取值、传递值给函数/基本运算(primitive operation)或从函数/基本运算中返回值。
  
* 错误的使用内联汇编，具体细节，参见使用内联汇编编写代码时的相关[规则][rules]。
* **在[常量上下文](const_eval.md#const-context)中**: 将指向某些以分配对象的指针（引用、原始指针或函数指针）转换或以其他方式重新解释为非指针类型（如整数）。
“重新解释”是指在不进行强制转换的情况下以整数类型加载指针值，例如通过执行原始指针强制转换或使用联合体（union）。

> **注意**：未定义行为影响整个程序。例如，在 C 中调用一个 C函数已经出现了未定义行为，这意味着包含此调用的整个程序都包含了未定义行为。如果 Rust 再通过 FFI 来调用这段 C程序/代码，那这段 Rust 代码也包含了未定义行为。反之亦然。因此 Rust 中的未定义行为会对任何其他通过 FFI 过来调用的代码造成不利影响。

### Pointed-to bytes
### 指向字节数据

指针或引用“指向”的字节数据是由指针值和指针对象类型的内存宽度（使用`size_of_val`）来确定的。

### Places based on misaligned pointers
### 基于未对齐指针的地址[based on a misaligned pointer]: #places-based-on-misaligned-pointers

如果地址计算过程中的最后一个`*`操作是在未按其类型对齐的指针上执行的，则称地址“基于未对齐的指针”。（如果位置表达式中没有`*`操作，则是访问局部变量的成员字段或访问静态变量(`static`)，rustc将确保适当的对齐方式。如果有多个`*`，则每个 `*`操作都会导致指针被从内存中被解引用出来，并且每个`*`操作都受对齐约束。请注意，由于自动解引用的存在，在 Rust语法中可以省略一些 `*`操作；我们在这里考虑的是全展开形式的位置表达式。）

例如，如果 `ptr` 的类型为 `*const S`，其中 `S` 的对齐方式为 8，则 `ptr` 必须是 8位对齐的，否则 `(*ptr).f` 就是“基于未对齐的指针”。
即使字段 `f` 的类型是 `u8`（诸如此类对齐量为1的类型），这个要求也是必须的。换句话说，对齐要求是源于被解引用的指针的类型，而不是正在访问的字段的类型。

请注意，只有在加载或存储到基于未对齐指针的地址时才会导致未定义行为。在基于未对齐指针执行 `&raw const`/`&raw mut` 是允许的。在一个地址上执行 `&`/`&mut`操作需要此地址按变量的字段类型进行对齐（否则程序将“产生非法值”），这通常是一个比基于对齐指针的限制更少的要求。如果字段类型可能比包含它的类型（例如 `repr(packed`修饰的变量的字段）更需要对齐，则将导致编译器报错。这意味着基于对齐的指针总是足以确保在其上出现的新引用总是对齐的，虽然这并不总是必要的。

### Dangling pointers
### 悬垂指针
[dangling]: #dangling-pointers

如果引用/指针[指向][points to]的所有字节不是同一个热分配(live allocation)或同一个热分配的一部分，那么它就是“悬垂(dangling)”的。

如果内存宽带为0，则指针通常从不“悬垂”（即使它是空指针）。

请注意，动态内存宽度类型（如切片和字符串）指向其底层的整个数据范围，因此他们的代表长度的元数据永远不要太大，这一点很重要。特别需要注意的是 Rust里，值的动态内存宽度（由`size_of_val` 来确定）不能超过 `isize::MAX`，因为单次的内存分配不可能分配出大于 `isize::MAX` 的内存宽度。

### Invalid values
### 非法值
[invalid-values]: #invalid-values

Rust编译器假设在程序执行期间生成的所有值都是“合法的”，因此生成非法值是立即UB。

值是否合法取决于它们的类型：
  * 布尔[`bool`]值必须是 `false` (`0`) 或 `true` (`1`)。
  * 函数指针(`fn` pointer)类型的值必须非空。
  * `char`值不能是代理码点(Surrogate)（即不能在字符区间`0xD800..=0xDFFF` 内），并且必须等于或小于`char::MAX`。
  * !`类型的值不能存在。
  * 整型值（`i*`/`u*`）、浮点值（`f*`）或裸指针必须初始化，即不能从[未初始化内存][undef]中获得。
  * `str`值被视为`[u8]`，即它必须被初始化。
  * `enum` 必须具有有效的判别值，并且由该判别值标示的变体的所有字段在其各自的类型下都必须合法。
  * `struct`、元组和数组要求所有字段/元素在其各自的类型下必须合法。
  * 对于 `union`，确切的合法性要求尚未确定。
    显然，所有可以完全在安全代码中创建的值都是合法的。
    如果联合体具有内存宽度为零的字段，则任何值都合法。
    进一步的细节[仍在争论中](https://github.com/rust-lang/unsafe-code-guidelines/issues/438)。
  * 引用或 [`Box<T>`] 必须对齐，不能[悬垂][dangling]，必须指向合法值（对于动态内存宽度的类型，使用由元数据确定的指针对象的实际动态类型）。
    请注意，最后一点（关于指向合法值）仍然未尽事宜存在一些争论。
  * 胖引用、[`Box<T>`] 或裸指针的元数据必须与类型尾部那个不定内存宽度的类型相匹配：
    * `dyn Trait`元数据必须是指向编译器为`Trait`生成的 vtable的指针。
      （对于裸指针，此要求仍然存在一些争论。）
    * 切片（`[T]`）的元数据必须是有效的 `usize`。
      此外，对于胖引用和 [`Box<T>`]，如果切片的元数据使指向值的总内存宽度大于`isize:：MAX`，则切片元数据无效。
  * 如果类型具有自定义的有效值区间，则合法值必须在该区间内。
    在标准库中，这条导致了 [`NonNull<T>`] 和 [`NonZero*`] 的出现。
    > **注意**：`rustc` 是使用还未稳定下来的属性 `rustc_layout_scalar_valid_range_*` 来达成这个目标的。

**注意：** 未初始化的内存对于任何具有有限合法值集的类型来说也隐式非法。也就是说，允许读取未初始化内存的情况只发生在联合体(`union`)内部和“对齐填充区(padding)”里（类型的字段/元素之间的间隙）。

[dangling]: #dangling-pointers
[`bool`]: types/boolean.md
[`const`]: items/constant-items.md
[noalias]: http://llvm.org/docs/LangRef.html#noalias
[pointer aliasing rules]: http://llvm.org/docs/LangRef.html#pointer-aliasing-rules
[undef]: http://llvm.org/docs/LangRef.html#undefined-values
[`target_feature`]: attributes/codegen.md#the-target_feature-attribute
[`UnsafeCell<U>`]: std::cell::UnsafeCell
[Rustonomicon]: ../nomicon/index.html
[`NonNull<T>`]: core::ptr::NonNull
[`NonZero<T>`]: core::num::NonZero
[place expression context]: expressions.md#place-expressions-and-value-expressions
[rules]: inline-assembly.md#rules-for-inline-assembly
[points to]: #pointed-to-bytes
[pointed to]: #pointed-to-bytes
[project-field]: expressions/field-expr.md
[project-tuple]: expressions/tuple-expr.md#tuple-indexing-expressions
[project-slice]: expressions/array-expr.md#array-and-slice-indexing-expressions
[const-promoted]: destructors.md#constant-promotion