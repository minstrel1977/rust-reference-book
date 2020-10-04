# Type parameters
# 类型参数

>[parameters.md](https://github.com/rust-lang/reference/blob/master/src/types/parameters.md)\
>commit eb02dd5194a747277bfa46b0185d1f5c248f177b

在带有类型参数声明的数据项的代码体内，其类型参数的名称是类型：

```rust
fn to_vec<A: Clone>(xs: &[A]) -> Vec<A> {
    if xs.is_empty() {
        return vec![];
    }
    let first: A = xs[0].clone();
    let mut rest: Vec<A> = to_vec(&xs[1..]);
    rest.insert(0, first);
    rest
}
```

这里，`first` 的类型为 `A`，引用的是 `to_vec` 的类型参数 `A`；`rest` 的类型为 `Vec<A>`，它是一个带有元素类型为 `A` vector。