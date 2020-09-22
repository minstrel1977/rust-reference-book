# 派生

>[derive.md](https://github.com/rust-lang/reference/blob/master/src/attributes/derive.md)\
>commit a52543267554541a95088b79f46a8bd36f487603

*`derive`属性*允许为结构数据自动生成新的[数据项]。它使用[_MetaListPaths_]句法规则指定要实现的 trait列表或相关的[派生宏]要处理的路径。

例如，下面将为 `Foo` 创建一个 [`PartialEq`] 和 [`Clone`] 这两个 trait 的 [`impl`数据项]，类型参数 `T` 将被相应的 `impl` 提供 `PartialEq` 或 `Clone` 约束：

```rust
#[derive(PartialEq, Clone)]
struct Foo<T> {
    a: i32,
    b: T,
}
```

上面代码为 `PartialEq` 生成的 `impl` 等价于

```rust
# struct Foo<T> { a: i32, b: T }
impl<T: PartialEq> PartialEq for Foo<T> {
    fn eq(&self, other: &Foo<T>) -> bool {
        self.a == other.a && self.b == other.b
    }

    fn ne(&self, other: &Foo<T>) -> bool {
        self.a != other.a || self.b != other.b
    }
}
```

可以通过[过程宏]为自定义的 trait 实现 `derive`。

## `automatically_derived`属性

*`automatically_derived`属性*被自动添加到由内置 trait 的 `derive`属性创建的[实现]中。它没有直接影响，但是工具和诊断lint 可以使用它来检测这些自动生成的实现。

[_MetaListPaths_]: ../attributes.md#元数据项属性句法
[`Clone`]: https://doc.rust-lang.org/std/clone/trait.Clone.html
[`PartialEq`]: https://doc.rust-lang.org/std/cmp/trait.PartialEq.html
[`impl` item]: ../items/implementations.md
[items]: ../items.md
[derive macros]: ../procedural-macros.md#派生宏
[implementations]: ../items/implementations.md
[items]: ../items.md
[procedural macros]: ../procedural-macros.md#派生宏
