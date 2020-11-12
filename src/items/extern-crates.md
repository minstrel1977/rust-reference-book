# Extern crate declarations
# 外部crate声明

>[extern-crates.md](https://github.com/rust-lang/reference/blob/master/src/items/extern-crates.md)\
>commit: afcf00bbf22f4dec71866a8bd5f385ba1d3b70af \
>本章译文最后维护日期：2020-11-8

> **<sup>句法:<sup>**\
> _ExternCrate_ :\
> &nbsp;&nbsp; `extern` `crate` _CrateRef_ _AsClause_<sup>?</sup> `;`
>
> _CrateRef_ :\
> &nbsp;&nbsp; [IDENTIFIER] | `self`
>
> _AsClause_ :\
> &nbsp;&nbsp; `as` ( [IDENTIFIER] | `_` )

*外部crate(`extern crate`)声明*指定了对外部 crate 的依赖关系。（这种声明让）外部的 crate 作为外部crate(`extern crate`)声明中提供的[标识符][identifier]被绑定到当前声明的作用域中。`as`子句可用于将导入的 crate 绑定到不同的名称上。

外部crate 在编译时被解析为一个特定的 `soname`[^soname]， 并且一个到此 `soname` 的运行时链接会传递给链接器，以便在运行时加载此 `soname`。`soname` 在编译时解析，方法是扫描编译器的库文件路径，匹配外部crate 的 `crateid`。因为`crateid` 是在编译时通过可选的 `crateid`属性声明的，所以如果外部 crate 没有提供 `crateid`， 则默认拿该外部crate 的 `name`属性值来和外部crate(`extern crate`)声明中的[标识符]绑定。

导入 `self` crate 会创建到当前 crate 的绑定。在这种情况下，必须使用 `as`子句指定要绑定到的名称。

三种外部crate(`extern crate`)声明的示例:

<!-- ignore: requires external crates -->
```rust,ignore
extern crate pcre;

extern crate std; // 等同于: extern crate std as std;

extern crate std as ruststd; // 使用其他名字去链接 'std'
```

当命名 Rust crate 时，不允许使用连字符(`-`)。然而 Cargo 包却可以使用它们。在这种情况下，当 `Cargo.toml` 文件中没有指定 crate 名称时， Cargo 将透明地将 `-` 替换为 `_` 以供 Rust 源文件内的外部crate(`extern crate`)声明引用 (详见 [RFC 940])。

这有一个示例：

<!-- ignore: requires external crates -->
```rust,ignore
// 导入 Cargo 包 hello-world
extern crate hello_world; // 连字符被替换为下划线
```

## Extern Prelude
## 外部预导入包

在根模块的源码中使用外部crate(`extern crate`)声明或者给编译命令增加编译参数（`rustc` 下使用 `--extern` 参数选项），这两种导入外部的 crate 的方式都是把外部的 crate 导入到“外部预导入包”里。外部预导入包里的 crate 的有效作用域是整个 crate，包括内部模块。如果使用 `extern crate orig_name as new_name` 导入，则相应地 symbol `new_name` 被添加到在预导入包里。

`core` crate 总是会被添加到外部预导入包里。只要没有在 crate 根模块中指定应用 [`no_std`]属性，也会把 `std` crate 添加进来。

可以在模块上使用 [`no_implicit_prelude`]属性来禁用模块内的预导入包查找功能。

> **版本差异**：在 2015 版中，在外部预导入包中的 crate 不能通过 [use声明][use declarations]来直接引用，因此通常标准做法是用外部crate(`extern crate`)将那它们纳入到当前作用域。
> 
> 从 2018 版开始， [use声明][use declarations]可以直接引用外部预导入包里的 crate，所以再使用外部crate(`extern crate`)就被认为是不规范的。

> **注意**: 随 `rustc` 一起引入的 crate，如 [`alloc`] 和 [`test`]，在使用 Cargo 时不会自动被包含在 `--extern` 命令行参数选项中。即使在 2018 版中，也必须通过外部crate(`extern crate`)声明来把它们引入到当前作用域内。
>
> ```rust
> extern crate alloc;
> use alloc::rc::Rc;
> ```

<!--
查看 https://github.com/rust-lang/rust/issues/57288 以了解更多关于 alloc/test 的限制。
-->
## Underscore Imports
## 下划线导入

外部的 crate依赖可以通过使用带有下划线形如 `extern crate foo as _` 的形式来声明，而无需将其名称绑定到当前作用域内。这种声明方式对于只需要 crate 被链接进来，但 crate 从不会被当前代码引用的情况可能很有用，并且还可以避免未使用的 lint 提醒。

下划线导入不会影响 [`macro_use`属性][`macro_use` attribute]的正常使用，这情况下使用 `macro_use`属性，宏名称仍会正常导入到 macro-use预导入包中。

## The `no_link` attribute
## `no_link`属性

可以在外部项(`extern crate` item)上指定使用 *`no_link`属性*，以防止此 crate 被链接到编译输出中。这通常用于加载一个 crate 而只访问它的宏。

[^soname]:译者注：这里的 `soname` 是 linux系统里的动态库文件的 soname。

[IDENTIFIER]: ../identifiers.md
[RFC 940]: https://github.com/rust-lang/rfcs/blob/master/text/0940-hyphens-considered-harmful.md
[`macro_use` attribute]: ../macros-by-example.md#the-macro_use-attribute
[`alloc`]: https://doc.rust-lang.org/alloc/
[`no_implicit_prelude`]: modules.md#prelude-items
[`no_std`]: ../crates-and-source-files.md#preludes-and-no_std
[`test`]: https://doc.rust-lang.org/test/
[use declarations]: use-declarations.md


<!-- 2020-11-12-->
<!-- checked -->
