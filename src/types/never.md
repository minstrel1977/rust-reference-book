# Never type
# never类型

>[never.md](https://github.com/rust-lang/reference/blob/master/src/types/never.md)\
>commit: 2d83af6a785893551d6fbe6235f1ffe66bea9eca \
>本章译文最后维护日期：2024-08-18

> **<sup>句法</sup>**\
> _NeverType_ : `!`

never类型(`!`)是一个没有值的类型，表示永远不会完成计算的结果。`!` 的类型表达式可以强转为任何其他类型。

类型`!` 目前**只能**出现在函数返回类型中，这表明它是一个从不真正返回的发散函数。

```rust
fn foo() -> ! {
    panic!("This call never returns.");
}
```

```rust
unsafe extern "C" {
    pub safe fn no_return_extern_func() -> !;
}
```
