# 模块

>[modules.md](https://github.com/rust-lang/reference/blob/master/src/items/modules.md)\
>commit f8e76ee9368f498f7f044c719de68c7d95da9972


> **<sup>句法:</sup>**\
> _Module_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `mod` [IDENTIFIER] `;`\
> &nbsp;&nbsp; | `mod` [IDENTIFIER] `{`\
> &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp; [_InnerAttribute_]<sup>\*</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp; [_Item_]<sup>\*</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp; `}`

模块是零个或多个[数据项]的容器。

*模块项*是一个用大括号括起来的，有命名的，并以关键字 `mod`作为前缀的模块。模块项将一个新的命名模块引入到组成 crate 的模块树中。模块可以任意嵌套。

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

模块和类型共享相同的命名空间。禁止在作用域中声明与模块同名的命名类型：也就是说，类型定义、trait、结构体、枚举、联合体、类型参数或 crate 不能在作用域中遮蔽模块的名称，反之亦然。使用 `use` 引入到当前作用域的数据项也受这个限制。

## 模块的源文件名

没有代码体的模块是从外部文件加载的。当模块没有 `path` 属性时，文件的路径和逻辑上的[模块路径]互为镜像。祖先模块路径组件是目录，模块的内容在一个文件中，该文件的名称是该模块名加上 `.rs` 的扩展。例如，下面的示例可以反映这种模块结构和文件系统结构相互映射的关系：

模块路径               | 文件系统路径  | 文件内容
------------------------- | ---------------  | -------------
`crate`                   | `lib.rs`         | `mod util;`
`crate::util`             | `util.rs`        | `mod config;`
`crate::util::config`     | `util/config.rs` |

当一个目录下有一个名为 `mod.rs` 的源文件时，模块的文件名也可以和这个目录互相映射。上面的例子也可以用一个承载同一源码内容的名为 `util/mod.rs`的实体文件来表达模块路径 `crate::util`。注意这两种表达方式都是不允许的 `util.rs` 和 `util/mod.rs` 同时存在。

> **注意**: 在`rustc` 1.30 之前，使用文件 `mod.rs` 是加载嵌套子模块的方法。现在鼓励使用新的命名约定，因为它更一致，并且可以避免在项目中搞出许多名为 `mod.rs` 的文件。
> 
### `path` 属性

用于加载外部文件模块的目录和文件可以受 `path` 属性的影响。

对于不在内联模块体内的模块上的`path` 属性，此属性引入的文件的路径为相对于源文件所在的目录。例如，下面的代码片段将使用基于其所在位置的路径:

<!-- ignore: requires external files -->
```rust,ignore
#[path = "foo.rs"]
mod c;
```

Source File    | `c`'s File Location | `c`'s Module Path
-------------- | ------------------- | ----------------------
`src/a/b.rs`   | `src/a/foo.rs`      | `crate::a::b::c`
`src/a/mod.rs` | `src/a/foo.rs`      | `crate::a::c`

对于内联模块体中的 `path` 属性，此属性引入的文件的路径的相对位置取决于 `path` 属性所在的源文件的类型。（我们知道）“mod-rs”源文件是根模块(如`lib.rs` 或 `main.rs`)和文件名为 `mod.rs` 的模块，非“mod-rs”源文件是所有其他模块文件。（那）在 mod-rs 文件中，内联模块体内的 `path` 属性（引入的文件的）路径相对于 mod-rs 文件的目录（该目录包括作为目录的内联模块组件名）。对于非 mod-rs 文件，除了路径以非 mod-rs 模块名称的目录开始外，情况也是一样的。例如，下面的代码片段将使用基于其所在位置的路径:

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

在内联模块上和内嵌模块内结合应用上述 `path` 属性规则的一个例子(同时适用于 mod-rs 和非 mod-rs 文件)：

<!-- ignore: requires external files -->
```rust,ignore
#[path = "thread_files"]
mod thread {
    // 从相对于当前源文件的目录下的 `thread_files/tls.rs` 文件里载 `local_data` 模块。
    #[path = "tls.rs"]
    mod local_data;
}
```

## 预导入包项

模块在作用域中隐式地有自己的名称。这些名称是内置类型，宏在外部 crate 上用[`#[macro_use]`][macro_use]导入这些名称，其他 crate 通过[预导入包]导入。这些名称都由唯一的标识符组成。这些名称不是当前模块的一部分，因此，例如，任何名为 `name`、 `self::name` 的路径都不是有效路径。通过[预导入包]添加的这种模块名称可以通过将 `no_implicit_prelude` [属性]放在当前模块或任意祖先模块上来删除。

## 模块上的属性

模块和所有数据项一样，接受外部属性。它们还接受内部属性：对于带有代码体的模块，在一个模块声明的 `{` 之后或者在模块源文件的开头（须在可选的 BOM 和 shebang 之后）都行。

在模块中有意义的内置属性是[`cfg`]、[`deprecated`]、[`doc`]、[lint检查类属性]、`path` 和 `no_implicit_prelude`。模块也接受宏属性。

[_InnerAttribute_]: ../attributes.md
[_Item_]: ../items.md
[macro_use]: ../macros-by-example.md#the-macro_use-attribute
[`cfg`]: ../conditional-compilation.md
[`deprecated`]: ../attributes/diagnostics.md#the-deprecated-attribute
[`doc`]: ../../rustdoc/the-doc-attribute.html
[IDENTIFIER]: ../identifiers.md
[属性]: ../attributes.md
[数据项]: ../items.md
[模块路径]: ../paths.md
[预导入包]: ../crates-and-source-files.md#preludes-and-no_std
[lint检查类属性]: ../attributes/diagnostics.md#lint-check-attributes
