# Static items
# 静态项

>[static-items.md](https://github.com/rust-lang/reference/blob/master/src/items/static-items.md)\
>commit 2f459e22ec30a94bafafe417da4e95044578df73

> **<sup>句法</sup>**\
> _StaticItem_ :\
> &nbsp;&nbsp; `static` `mut`<sup>?</sup> [IDENTIFIER] `:` [_Type_]
>              `=` [_Expression_] `;`

*静态项*类似于[常量]，除了它在程序中表示一个精确的内存位置。所有对静态项的引用都指向相同的内存位置。静态项有 `static` 生存期，它比 Rust 程序中的所有其他（数据项）生存期都要长。静态项不会在程序结束时调用 [`drop`]。

静态初始化器是在编译时计算的[常量表达式]。静态初始化器可以引用其他静态项。

包含非[内部可变]类型的非 `mut` 静态项可以放在只读内存中。

所有访问静态项的操作都是安全的，但对静态项也有一些限制：

* 静态项的数据类型必须拥有 `Sync` trait，这样才可以让线程安全访问。
* 常量项不能引用静态项。

## Mutable statics
## 可变静态项

如果静态项是用 `mut` 关键字声明的，则程序允许对其进行修改。Rust 的目标之一是避免并发带来的 bug，可变静态项显然是竞争条件或其他 bug 的一个非常重要的来源。因此，读取或写入可变静态项变量时需要 `unsafe` 块。应注意确保对可变静态项的修改相对于运行在同一进程中的其他线程是安全的。

然而，可变静态项仍然非常有用。它们可以与 C 库一起使用，也可以在 `extern` 块中与 C 库绑定。

```rust
# fn atomic_add(_: &mut u32, _: u32) -> u32 { 2 }

static mut LEVELS: u32 = 0;

// 这违反了不共享状态的思想，而且它在内部不能防止竞争，所以这个函数是 `unsafe`
unsafe fn bump_levels_unsafe1() -> u32 {
    let ret = LEVELS;
    LEVELS += 1;
    return ret;
}

// 假设我们有一个返回旧值的 atomic_add 函数，这个函数是“安全的”，
// 但是返回值的含义可能不是调用者所期望的，所以它仍然被标记为 `unsafe`
unsafe fn bump_levels_unsafe2() -> u32 {
    return atomic_add(&mut LEVELS, 1);
}
```

可变静态项与普通静态项具有相同的限制，除了可变静态项的类型不需要实现 `Sync` trait。

## Using Statics or Consts
## 使用常量项或静态项

是否应该使用常量项还是静态项可能会令人困惑。一般来说，常量项应优先于静态项，除非以下情况之一成立：

* 存储大量数据
* 需要静态项的存储地址不变的特性。
* 需要内部可变性。

[常量]: constant-items.md
[`drop`]: ../destructors.md
[常量表达式]: ../const_eval.md#常量表达式
[内部可变]: ../interior-mutability.md
[IDENTIFIER]: ../identifiers.md
[_Type_]: ../types.md#type-expressions
[_Expression_]: ../expressions.md
