# Static items
# 静态项

>[static-items.md](https://github.com/rust-lang/reference/blob/master/src/items/static-items.md)\
>commit: 060d2b35cecf93bfd7fa20faa6c7f9b6aa1bc89f \
>本章译文最后维护日期：2024-10-13

> **<sup>句法</sup>**\
> _StaticItem_ :\
> &nbsp;&nbsp; [_ItemSafety_]<sup>?</sup>[^extern-safety] `static` `mut`<sup>?</sup> [IDENTIFIER] `:` [_Type_]
>              ( `=` [_Expression_] )<sup>?</sup> `;`
>
> [^extern-safety]: `safe` 和 `unsafe` 函数限定符仅在 `extern`块中的语义上允许使用。

*静态项*类似于[常量项][constant]，除了它在程序中表示一个精确的内存位置。所有对静态项的引用都指向相同的内存位置。静态项拥有 `'static` 生存期，它比 Rust 程序中的所有其他生存期都要长。静态项不会在程序结束时调用析构动作 [`drop`]。

静态项的声明在其所在的模块或块的[值命名空间][value namespace]中定义静态值。

静态初始化器是在编译时求值的[常量表达式][constant expression]。静态初始化器可以引用其他静态项。

包含非[内部可变][interior mutable]类型的非 `mut` 静态项可以放在只读内存中。

所有访问静态项的操作都是安全的，但对静态项有一些限制：

* 静态项的数据类型必须有 `Sync` trait约束，这样才可以让线程安全地访问。
* 常量项不能引用静态项。

必须为自由静态项提供初始化表达式，但在[外部块][external block]中静态项必须省略初始化表达式。

仅当在[外部块][external block]中使用时，语义上才允许使用`safe`和`unsafe`限定符。

## Statics & generics
## 静态项和泛型
在泛型作用域中（例如在包覆实现或默认实现中）定义的静态项将只会定义一个静态项，就好像该静态项定义是从当前作用域中拉入到模块内一样。*不会*出现程序项在单态化的过程中各自出现自己的独有静态项。

例如：

```rust
use std::sync::atomic::{AtomicUsize, Ordering};

trait Tr {
    fn default_impl() {
        static COUNTER: AtomicUsize = AtomicUsize::new(0);
        println!("default_impl: counter was {}", COUNTER.fetch_add(1, Ordering::Relaxed));
    }

    fn blanket_impl();
}

struct Ty1 {}
struct Ty2 {}

impl<T> Tr for T {
    fn blanket_impl() {
        static COUNTER: AtomicUsize = AtomicUsize::new(0);
        println!("blanket_impl: counter was {}", COUNTER.fetch_add(1, Ordering::Relaxed));
    }
}

fn main() {
    <Ty1 as Tr>::default_impl();
    <Ty2 as Tr>::default_impl();
    <Ty1 as Tr>::blanket_impl();
    <Ty2 as Tr>::blanket_impl();
}
```

以上会打印如下：

```text
default_impl: counter was 0
default_impl: counter was 1
blanket_impl: counter was 0
blanket_impl: counter was 1
```

## Mutable statics
## 可变静态项

如果静态项是用关键字 `mut` 声明的，则它允许被程序修改。Rust 的目标之一是使尽可能的避免出现并发 bug，那允许修改可变静态项显然是竞态(race conditions)或其他 bug 的一个重要来源。因此，读取或写入可变静态项变量时需要引入非安全(`unsafe`)块。应注意确保对可变静态项的修改相对于运行在同一进程中的其他线程来说是安全的。

尽管有这些缺点，可变静态项仍然非常有用。它们可以与 C库一起使用，也可以在外部(`extern`)块中从 C库中来绑定它。

```rust
# fn atomic_add(_: *mut u32, _: u32) -> u32 { 2 }

static mut LEVELS: u32 = 0;

// 这违反了不共享状态的思想，而且它在内部不能防止竞争，所以这个函数是非安全的(`unsafe`)
unsafe fn bump_levels_unsafe() -> u32 {
    unsafe {
        let ret = LEVELS;
        LEVELS += 1;
        return ret;
    }
}

// 作为 `bump_levels_unsafe` 的替代方法，该函数是安全的，假设我们有一个返回旧值的 atomic_add函数。只有当没有其他代码以非原子方式访问静态项时，该函数才是安全的。如果可以进行这样的访问（例如在 `bump_levels_unsafe` 中），则这需把此函数标记为 `unsafe'，以向调用者表达出它们仍然必须防止并发访问的意思。
fn bump_levels_safe() -> u32 {
    unsafe {
        return atomic_add(&raw mut LEVELS, 1);
    }
}
```

除了可变静态项的类型不需要实现 `Sync` trait 之外，可变静态项与普通静态项具有相同的限制。

## Using Statics or Consts
## 使用常量项或静态项

应该使用常量项还是应该使用静态项可能会令人困惑。一般来说，常量项应优先于静态项，除非以下情况之一成立：

* 存储大量数据
* 需要静态项的存储地址不变的特性。
* 需要内部可变性。

[constant]: constant-items.md
[`drop`]: ../destructors.md
[constant expression]: ../const_eval.md#constant-expressions
[external block]: external-blocks.md
[interior mutable]: ../interior-mutability.md
[IDENTIFIER]: ../identifiers.md
[_Type_]: ../types.md#type-expressions
[_Expression_]: ../expressions.md
[value namespace]: ../names/namespaces.md
[_ItemSafety_]: functions.md
