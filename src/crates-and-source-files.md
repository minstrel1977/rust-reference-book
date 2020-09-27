# crate 和源文件

>[crates-and-source-files.md](https://github.com/rust-lang/reference/blob/master/src/crates-and-source-files.md)\
>commit 277587a55aa24d8f6a66ddb43493e150c916ef43

> **<sup>句法</sup>**\
> _Crate_ :\
> &nbsp;&nbsp; UTF8BOM<sup>?</sup>\
> &nbsp;&nbsp; SHEBANG<sup>?</sup>\
> &nbsp;&nbsp; [_InnerAttribute_]<sup>\*</sup>\
> &nbsp;&nbsp; [_Item_]<sup>\*</sup>

> **<sup>词法</sup>**\
> UTF8BOM : `\uFEFF`\
> SHEBANG : `#!` \~`\n`<sup>\+</sup>[†](#shebang)


> 注意：尽管像任何其他语言一样，Rust 可以通过解释器和编译器实现，但现在唯一存在的实现是编译器，而且该语言一直为可编译而被设计的。鉴于这些原因，本章节设定为走编译器这条路径。

Rust 的语义遵循编译时和运行时之间的*阶段区别*。[^phase-distinction] 其中*静态解释*的语义规则控制编译的成功或失败，而*动态解释*的语义规则控制程序在运行时的行为。

编译模型以称为 _crate_ 的构件为中心。每次编译都以源码的形式处理单个的 crate，如果成功，将生成二进制形式的单个 crate：可执行文件或某种类型的库文件。[^cratesourcefile]

crate 是编译和链接的单元，也是版本控制、发布和运行时加载的基本单元。一个 crate 包含一个嵌套的[模块]作用域*树*。这个树的顶层是一个匿名的模块(从模块内部路径的角度来看)，并且一个 crate 中的任何数据项都有一个规范的[模块路径]，表示它在 crate 的模块树中的位置。

Rust 编译器总是使用单个源文件作为输入来调用，并且总是生成单个输出 crate。对输入源文件的处理可能导致其他源文件作为模块被加载进来。源文件的扩展名为 `.rs`。

Rust源文件描述了一个模块，其名称和位置（在当前 crate 的模块树中）是从源文件外部定义的：要么通过引用源文件中的显式[_Module_][模块]项，要么由 crate 本身的名称定义。每个源文件都是一个模块，但并非每个模块都需要自己的源文件：[模块定义][模块]可以嵌套在一个文件中。

每个源文件包含一个由零个或多个[*数据项*]定义组成的序列，并且可选地从应用于包含模块的任意数量的[属性]开始，这些[属性]中的大部分会影响编译器的行为。匿名 crate 模块可以具有应用于整个 crate 的附加属性。

```rust
// 指定 crate 名称.
#![crate_name = "projx"]

// 指定编译输出文件的类型
#![crate_type = "lib"]

// 打开一种警告
// 这句可以放在任何模块中, 而不是只能放在匿名 crate 模块里。
#![warn(non_camel_case_types)]
```

## 字节序标记

可选的[_utf8 字节序标记_](由 UTF8BOM 生成)表示该文件是用 UTF8 编码的。它只能出现在文件的开头，并且编译器会忽略它。

## Shebang

源文件可以有一个[_shebang_]（由 SHEBANG 生成），它指示操作系统使用什么程序来执行此文件。它本质上是将源文件作为可执行脚本处理。shebang 只能出现在文件的开头(但是在可选的 _UTF8BOM_ 之后)。它会被编译器忽略。例如：

<!-- ignore: tests don't like shebang -->
```rust,ignore
#!/usr/bin/env rustx

fn main() {
    println!("Hello!");
}
```

为了避免与[属性]混淆， Rust 对 shebang 语法做了一个限制， 是 `#!` 字符不能后跟`[` 标记码，忽略中间的[注释]或[空白]。如果此限制失败，则不将其视为 shebang，而将其视为属性的开始。

## 预导入包和 `no_std`

所有的 crate 都有一个 *预导入包*，它会自动将一个特定模块（*预导入包模块*）的名称插入到每个[模块]的作用域内，并将一个 [`extern crate`] 插入到 crate 的根模块中。默认情况下，这个特定的模块为*标准预导入包* 。链接的 crate 是 [`std`]，预导入包模块是 [`std::prelude::v1`]。

在根 crate 模块上使用 `no_std` [属性]，可以将预导入包改成 *核心预导入包*。连接的板条箱为 [`core`]，预导入模块为 [`core::prelude::v1`]。当 crate 的目标平台不支持标准库或有意不使用标准库的功能时，使用核心预导入包而不是标准预导入包是有用的。这么做放弃的主要功能是动态内存分配(例如： `Box` 和 `Vec`)、文件和网络功能(例如： `std::fs` and `std::io`)。

<div class="warning">

警告:使用 `no_std` 不会阻止主动去链接标准库。仍然可以将 `extern crate std;` 这条语句放入到 crate 中，这样完整依赖关系也可以成功加载。

</div>

## main函数

包含 `main` [函数]的 crate 可以被编译成可执行文件。如果一个 `main` 函数存在，它必须不能有参数，不能声明任何 [trait 或生命周期约束]，不能有任何 [where 子句]，并且它的返回类型必须是以下类型之一:

* `()`
* `Result<(), E> where E: Error`
<!-- * `!` -->
<!-- * Result<!, E> where E: Error` -->

> 注意: 允许哪些返回类型的实现是由暂未稳定的[`Termination`] trait 决定的。

<!-- 如果前面这节需要更新(从 "必须不能有参数" 开始, 同时需要修改 attributes/testing.md 文件 -->

### `no_main`属性

在 crate 层级，可应用*`no_main` [属性]*来禁止对可执行二进制文件发出 `main` 符号——即禁止当前crate的 main 函数的执行。当链接的其他对象定义了 `main` 时，这很有用。

## `crate_name`属性

在 crate 层级，可应用*`crate_name` [属性]*，配对使用 [_MetaNameValueStr_] 语法来指定 crate 的名称。

```rust
#![crate_name = "mycrate"]
```

crate 名称不能为空，只能包含[Unicode字母数字]或 `-` (U+002D)字符。

[^phase-distinction]: 这种区别也存在于解释器中。静态检查(如语法分析、类型检查和 lint )应该在程序执行之前进行，而不管它何时执行。

[^cratesourcefile]: crate 有点类似于 ECMA-335 CLI 模型中的 *assembly*、SML/NJ 编译管理器中的 *library*、Owens 和 Flatt 模块系统中的 *unit*， 或 Mesa 中的 *configuration*。

[Unicode alphanumeric]: ../std/primitive.char.html#method.is_alphanumeric
[_InnerAttribute_]: attributes.md
[_Item_]: items.md
[_MetaNameValueStr_]: attributes.md#元项属性句法
[_shebang_]: https://en.wikipedia.org/wiki/Shebang_(Unix)
[_utf8 字节序标记_]: https://en.wikipedia.org/wiki/Byte_order_mark#UTF-8
[`Termination`]: ../std/process/trait.Termination.html
[`core`]: ../core/index.html
[`core::prelude::v1`]: ../core/prelude/index.html
[`extern crate`]: items/extern-crates.md
[`std`]: ../std/index.html
[`std::prelude::v1`]: ../std/prelude/index.html
[属性]: attributes.md
[注释]: comments.md
[函数]: items/functions.md
[模块]: items/modules.md
[模块路径]: paths.md
[trait 或生命周期约束]: trait-bounds.md
[where 字句]: items/generics.md#where子句
[空白]: whitespace.md
