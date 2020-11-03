# Modules
# 模块

>[modules.md](https://github.com/rust-lang/reference/blob/master/src/items/modules.md)\
>commit: f8e76ee9368f498f7f044c719de68c7d95da9972 \
>本译文最后维护日期：2020-11-3


> **<sup>句法:</sup>**\
> _Module_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `unsafe`<sup>?</sup> `mod` [IDENTIFIER] `;`\
> &nbsp;&nbsp; | `unsafe`<sup>?</sup> `mod` [IDENTIFIER] `{`\
> &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp; [_InnerAttribute_]<sup>\*</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp; [_Item_]<sup>\*</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp; `}`

模块是零个或多个[数据项][items]的容器。

*模块项*是一个用花括号括起来的，有命名的，并以关键字 `mod` 作为前缀的模块。模块项将一个新的命名模块引入到组成 crate 的模块树中。模块可以任意嵌套。

模块的一个例子：

```rust
mod math {
    type Complex = (f64, f64);
    fn sin(f: f64) -> f64 {
        /* ... */
#       unimplemented!();
    }
    fn cos(f: f64) -> f64 {
        /* ... */
#       unimplemented!();
    }
    fn tan(f: f64) -> f64 {
        /* ... */
#       unimplemented!();
    }
}
```

模块和类型共享相同的命名空间。禁止在同一个作用域中声明与此作用域下模块同名的命名类型(named type)：也就是说，类型定义、trait、结构体、枚举、联合体、类型参数或 crate 不能在其作用域中屏蔽此作用域中也生效的模块名称，反之亦然。使用 `use` 引入到当前作用域的数据项也受这个限制。

在句法上，关键字 `unsafe` 允许出现在关键字 `mod`之前，但是在语义层面却会被弃用。这种设计允许宏在将关键字 `unsafe` 从标记流中移除之前利用此句法来使用此关键字。

## Module Source Filenames
## 模块的源文件名

没有代码体的模块是从外部文件加载的。当模块没有 `path` 属性限制时，文件的路径和逻辑上的[模块路径][module path]互为镜像。祖先模块的路径组件(path component)是此模块文件的目录，而模块的内容存在一个以该模块名为文件名，再加上 `.rs` 为扩展文件名的文件中。例如，下面的示例可以反映这种模块结构和文件系统结构相互映射的关系：

模块路径               | 文件系统路径  | 文件内容
------------------------- | ---------------  | -------------
`crate`                   | `lib.rs`         | `mod util;`
`crate::util`             | `util.rs`        | `mod config;`
`crate::util::config`     | `util/config.rs` |

当一个目录下有一个名为 `mod.rs` 的源文件时，模块的文件名也可以和这个目录互相映射。上面的例子也可以用一个承载同一源码内容的名为 `util/mod.rs` 的实体文件来表达模块路径 `crate::util`。注意不允许 `util.rs` 和 `util/mod.rs` 同时存在。

> **注意**：在`rustc` 1.30 版本之前，使用文件 `mod.rs` 是加载嵌套子模块的方法。现在鼓励使用新的命名约定，因为它更一致，并且可以避免在项目中搞出许多名为 `mod.rs` 的文件。

### The `path` attribute
### `path`属性

用于加载外部文件模块的目录和文件可以受 `path`属性的影响。

对于不在内联模块(inline module)块内的模块上的 `path`属性，此属性引入的文件的路径为相对于当前源文件所在的目录。例如，下面的代码片段将使用基于其所在位置的路径:

<!-- ignore: requires external files -->
```rust,ignore
#[path = "foo.rs"]
mod c;
```

Source File    | `c`'s File Location | `c`'s Module Path
-------------- | ------------------- | ----------------------
`src/a/b.rs`   | `src/a/foo.rs`      | `crate::a::b::c`
`src/a/mod.rs` | `src/a/foo.rs`      | `crate::a::c`

对于处在内联模块块内的 `path` 属性，此属性引入的文件的路径取决于 `path` 属性所在的源文件的类型。（先对源文件进行分类，）“mod-rs”源文件是根模块（比如是 `lib.rs` 或 `main.rs`）和文件名为 `mod.rs` 的模块，非“mod-rs”源文件是所有其他模块文件。（那）在 mod-rs 文件中，内联模块块内的 `path` 属性（引入的文件的）路径是相对于 mod-rs 文件的目录（该目录包括作为目录的内联模块组件名）。对于非 mod-rs 文件，除了路径以此模块名为目录前段外，其他是一样的。例如，下面的代码片段将使用基于其所在位置的路径：

<!-- ignore: requires external files -->
```rust,ignore
mod inline {
    #[path = "other.rs"]
    mod inner;
}
```

Source File    | `inner`'s File Location   | `inner`'s Module Path
-------------- | --------------------------| ----------------------------
`src/a/b.rs`   | `src/a/b/inline/other.rs` | `crate::a::b::inline::inner`
`src/a/mod.rs` | `src/a/inline/other.rs`   | `crate::a::inline::inner`

在内联模块上和内嵌模块内混合应用上述 `path` 属性规则的一个例子（mod-rs 和非 mod-rs 文件都适用）：
An example of combining the above rules of `path` attributes on inline modules
and nested modules within (applies to both mod-rs and non-mod-rs files):

<!-- ignore: requires external files -->
```rust,ignore
#[path = "thread_files"]
mod thread {
    // 从相对于当前源文件的目录下的 `thread_files/tls.rs` 文件里载 `local_data` 模块。
    #[path = "tls.rs"]
    mod local_data;
}
```

## Prelude Items
## 预导入项

模块在作用域中隐式地就有一些模块名称。这些名称是内置的，除了宏（在外部crate 上是）用 [`#[macro_use]`][macro_use] 导入这些名称，其他都是通过当前 crate 的[预导入包][prelude]导入的。这些名称都由唯一的标识符组成。这些名称不是当前模块的一部分，因此，例如，任何名为 `name`、 `self::name` 的路径都不是有效路径。由[预导入包][prelude]添加进来的各种模块名称可以通过将 `no_implicit_prelude`[属性][attribute]放在当前模块或当前模块的任意祖先模块上来移除。

## Attributes on Modules
## 模块上的属性

模块和所有数据项一样能接受外部属性。它们也能接受内部属性：可以在带有代码体的模块的 `{` 之后，也可以在模块源文件的开头（须在可选的 BOM 和 shebang 之后）。

在模块中有意义的内置属性是 [`cfg`]、[`deprecated`]、[`doc`]、[lint检查类属性][the lint check attributes]、`path` 和 `no_implicit_prelude`。模块也接受宏属性。

[_InnerAttribute_]: ../attributes.md
[_Item_]: ../items.md
[macro_use]: ../macros-by-example.md#the-macro_use-attribute
[`cfg`]: ../conditional-compilation.md
[`deprecated`]: ../attributes/diagnostics.md#the-deprecated-attribute
[`doc`]: https://doc.rust-lang.org/rustdoc/the-doc-attribute.html
[IDENTIFIER]: ../identifiers.md
[attribute]: ../attributes.md
[items]: ../items.md
[module path]: ../paths.md
[prelude]: ../crates-and-source-files.md#preludes-and-no_std
[the lint check attributes]: ../attributes/diagnostics.md#lint-check-attributes

<!-- 2020-11-3 -->
<!-- checked -->
