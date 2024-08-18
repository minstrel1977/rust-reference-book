# The `unsafe` keyword
# `unsafe`关键字

>[unsafe-keyword.md](https://github.com/rust-lang/reference/blob/master/src/unsafe-keyword.md)\
>commit:  875b905a389455c5329ae088600c0b5f7222104d \
>本章译文最后维护日期：2024-08-18

`unsafe`关键字可以出现在几个不同的上下文中：
unsafe函数(`unsafe fn`)、unsafe块(`unsafe {}`)、unsafe traits(`unsafe trait`)，unsafe实现(`unsafe impl`)，以及 unsafe外部块(`unsafe extern`)中。
根据它的使用位置以及是否启用了 `unsafe_op_in_unsafe_fn` lint，它扮演着几种不同的角色：
- 它用于标记*定义*额外安全条款要求（`unsafe fn`、`unsafe trait`）的代码
- 它用于标记需要*满足*额外安全条款(satisfy)要求的代码（`unsafe {}`、`unsafe impl`、不带[`unsafe_op_in_unsafe_fn`]的`unsafe fn`、`unsafe extern`）

接下来会讨论这里的每种情况。
参见[关键字相关文档][keyword]，那里有一些直观的例子。

## Unsafe functions (`unsafe fn`)
## unsafe函数(`unsafe fn`)

unsafe函数是指在所有上下文和/或所有可能的输入中可能不安全的函数。
我们说函数有*额外的安全条款要求*，这是所有调用者必须遵守的要求，并且编译器不对其进行检查。
例如，[`get_unchecked`]具有额外的安全条款要求，即索引必须在边界内。
unsafe函数应随附文档列明这些额外安全条款要求。

unsafe函数必须以 `unsafe`关键字为前缀，并且只能在 `unsafe`块内部去调用，或者在不带[`unsafe_op_in_unsafe_fn`] lint的 `unsafe fn` 内部去调用。

## Unsafe blocks (`unsafe {}`)

一块代码可以以 `unsafe`关键字为前缀，以允许调用 `unsafe`函数或解引用裸指针。
默认情况下，unsafe函数的主体也被视为 unsafe块；这可以通过启用 [`unsafe_op_in_unsafe_fn`] lint来改变这个默认。

通过将操作放入一个 unsafe块中，程序员声明确认他们已经注意到了满足该块中所有操作的额外安全条款要求。

unsafe块是 unsafe函数的逻辑对偶：

在 unsafe函数定义调用方必须秉承的证明义务(proof obligation)的情况下，unsafe块则声明在本块内调用的函数或操作的所有相关的证明义务都已解除(编译器认为相关的证明义务已经被程序员完成)。

履行举证义务有多种方式；例如，可以有运行时检查或通过结构化的不变式(invariant)来保证特定属性肯定为真，或者让 unsafe块放在 `unsafe fn` 内，这样，块就可以使用该函数的证明义务来解除(编译器对)从块内产生的（让程序员）履行证明义务的要求。

unsafe块用于包装外部库、直接使用硬件或实现当前语言中不直接存在的特性(features)。
例如，Rust 提供了在语言中实现内存安全并发所需的语言特性，但背后的事实是：标准库中线程和消息传递功能都是通过使用了 unsafe块来实现的。

Rust的类型系统是动态安全需求的保守近似实现，所以在某些情况下，使用安全代码会带来较大的性能开销。
例如，双链表不是树结构，只能用安全代码中的引用计数指针表示。
通过使用 `unsafe`块将反向链接表示为裸指针，它可以在不进行引用计数的情况下实现。
（请参阅["Learn Rust With Entirely Too Many Linked Lists"](https://rust-unofficial.github.io/too-many-lists/)来对这个特定示例进行更深入的探索。）

## Unsafe traits (`unsafe trait`)

unsafe trait 是一种带有额外安全条款要求的 trait，该trait 的*实现*必须要满足实现这些安全条款。
unsafe trait 应随附解释这些额外安全条款要求的文档。

这种 trait 必须以 `unsafe`关键字为前缀，并且只能由 `unsafe impl`块来实现。

## Unsafe trait implementations (`unsafe impl`)

当实现一个 unsafe trait 时，实现需要以 `unsafe`关键字作为前缀。
通过书写 `unsafe impl`，程序员表示他们已经注意到并满足了此trait 所需的额外安全条款要求。

unsafe trait实现是 unsafe trait 的逻辑对偶：在 unsafe trait 里定义了要实现本 trait 的实现必须秉承的证明义务，unsafe实现则申明了所有相关的证明义务（程序员）都已完成了（，编译器可以不用管这里的证明义务了）。

[keyword]: https://doc.rust-lang.org/std/keyword.unsafe.html
[`get_unchecked`]: https://doc.rust-lang.org/std/primitive.slice.html#method.get_unchecked
[`unsafe_op_in_unsafe_fn`]: https://doc.rust-lang.org/rustc/lints/listing/allowed-by-default.html#unsafe-op-in-unsafe-fn

## Unsafe external blocks (`unsafe extern`)
## Unsafe外部块(`unsafe extern`)

声明[外部块][external block]的程序员必须确保其中包含的程序项的签名是正确的。否则可能会导致未定义的行为。`unsafe extern`一词表明该义务已得到履行。

[external block]: items/external-blocks.md
