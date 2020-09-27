# 外部 crate 声明

>[extern-crates.md](https://github.com/rust-lang/reference/blob/master/src/items/extern-crates.md)\
>commit afcf00bbf22f4dec71866a8bd5f385ba1d3b70af

> **<sup>句法:<sup>**\
> _ExternCrate_ :\
> &nbsp;&nbsp; `extern` `crate` _CrateRef_ _AsClause_<sup>?</sup> `;`
>
> _CrateRef_ :\
> &nbsp;&nbsp; [IDENTIFIER] | `self`
>
> _AsClause_ :\
> &nbsp;&nbsp; `as` ( [IDENTIFIER] | `_` )

*`extern crate` 声明*指定对外部 crate 的依赖关系。（这种声明让）外部的 crate 作为 `extern crate` 声明中提供的[标识符]被绑定到当前声明的作用域中。`as` 子句可用于将导入的 crate 绑定到不同的名称上。
外部的 crate 在编译时被解析为一个特定的 `soname`[^soname]， 并且该 `soname` 的运行时链接被要求传递给链接器，以便在运行时加载。`soname` 在编译时解析，方法是扫描编译器的库路径，并匹配外部 crate 上（在它编译时）通过可选的 `crateid` 属性声明的`crateid`。如果外部 crate 没有提供 `crateid` ， 则（假想中的）默认的 `name` 属性值来和 `extern crate` 声明中的[标识符]绑定。

导入`self` crate 会创建到当前 crate 的一个绑定。在这种情况下，必须使用 `as` 子句指定要绑定到的名称。

三种 `extern crate` 声明的示例:

<!-- ignore: requires external crates -->
```rust,ignore
extern crate pcre;

extern crate std; // 等同于: extern crate std as std;

extern crate std as ruststd; // 使用其他名字去链接 'std'
```

当命名 Rust crate时，连字符是不允许的。然而 Cargo 包却可以使用它们。在这种情况下，当 `Cargo.toml` 没有指定 crate 名称时， Cargo 将透明地将（Rust 源文件中的 `extern crate` 声明中的标识符中的） `-` 替换为 `_` (详见[RFC 940])。

这有一个示例：

<!-- ignore: requires external crates -->
```rust,ignore
// 导入 Cargo 包 hello-world
extern crate hello_world; // 连字符被替换为下划线
```

## 外部预导入包

在根模块的源码中使用 `extern crate` 声明或者给编译命令增加编译参数(`rustc` 下使用 `--extern` 选项)这样导入外部的 crate 的方式是把外部的 crate 导入到“外部预导入包”里。外部预导入包里的 crate 的有效作用域是整个 crate，包括内部模块。如果使用 `extern crate orig_name as new_name` 导入，则标志符 `new_name` 会被替代着添加到在预导入包里。

`core` crate 总是会添加到外部预导入包里。只要没有在 crate 根中指定 [`no_std`] 属性，也会添加 `std` crate （到外部预导入包里）。

可以在模块上使用 [`no_implicit_prelude`] 属性来禁用模块内的预导入包查找。

> **版本差异**：在 2015 版中，在外部预导入包中的 crate 不能通过[use 声明]来引用，因此通常标准做法是用 `extern crate` 将那它们纳入到当前作用域。从 2018 版开始， [use 声明]可以引用外部预导入包里的 crate，所以再使用 `extern crate` 就被认为是不规范的。

> **注意**: `rustc` 附带的 crate，如 [`alloc`] 和 [`test`]，在使用 Cargo 时不会自动被包含在 `--extern` 选项中。即使在 2018 版中，也必须通过 `extern crate` 声明来把它们引入到当前作用域内。
>
> ```rust
> extern crate alloc;
> use alloc::rc::Rc;
> ```

<!--
See https://github.com/rust-lang/rust/issues/57288 for more about the alloc/test limitation.
-->

## 下划线导入

外部的 crate 依赖可以通过使用带有下划线形如 `extern crate foo as _` 的形式来声明，而无需将其名称绑定在作用域内。这对于只需要链接但从不引用的 crate 可能很有用，并且可以避免被提醒为未使用。

（下划线导入不会影响）[`macro_use` 属性]的正常工作，（此情况下）宏名称将会正常导入到 macro-use prelude 中。

## `no_link`属性

可以在 `extern crate` 数据项上指定 *`no_link` 属性*，以防止将 crate 链接到输出中。这通常用于加载一个 crate 而只访问它的宏。

[^soname]:译者注：这里故意使用soname是为了让读者主动联想类比linux系统里的动态库文件的 soname。

[IDENTIFIER]: ../identifiers.md
[RFC 940]: https://github.com/rust-lang/rfcs/blob/master/text/0940-hyphens-considered-harmful.md
[`macro_use` attribute]: ../macros-by-example.md#the-macro_use-attribute
[`alloc`]: https://doc.rust-lang.org/alloc/
[`no_implicit_prelude`]: modules.md#prelude-items
[`no_std`]: ../crates-and-source-files.md#preludes-and-no_std
[`test`]: https://doc.rust-lang.org/test/
[use declarations]: use-declarations.md
