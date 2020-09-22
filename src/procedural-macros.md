## 过程宏

>[procedural-macros.md](https://github.com/rust-lang/reference/blob/master/src/procedural-macros.md)\
>commit a1ef5a09c0281b0f2a65c18670e927ead61eb1b2

*过程宏*允许在执行函数时创建句法扩展。过程宏有三种形式:

* [类函数宏] - `custom!(...)`
* [派生宏]- `#[derive(CustomDerive)]`
* [属性宏] - `#[CustomAttribute]`

过程宏允许您在编译时运行对 Rust 句法进行操作的代码，同时使用并生成 Rust 句法。您可以将过程宏看作是从一个 <abbr title="抽象句法树：Abstract Syntax Tree">AST</abbr> 到另一个 <abbr title="抽象句法树：Abstract Syntax Tree">AST</abbr> 的函数。

过程宏必须在 [crate 类型]为 `proc-macro` 的 crate 中定义。

> **注意**: 使用 Cargo 时，定义过程宏的 crate 的配置文件里要有如下设置：
>
> ```toml
> [lib]
> proc-macro = true
> ```

作为函数，它们要么有返回句法、panic，要么永无休止地循环。返回的句法根据过程宏的类型替换或添加句法。编译器捕获 panic 并将其转化为编译器错误。编译器不会捕获死循环，而是会被挂起。

过程宏在编译期运行，因此具有与编译器相同的资源。例如，编译器可以访问的标准输入、错误和输出是相同的。类似地，文件访问也是一样的。因此，过程宏与 [Cargo的构建脚本] 具有相同的安全问题。

过程宏有两种报告错误的方法。首先是 panic；第二个是发出 [`compile_error`] 性质的宏调用。

### `proc_macro` crate

过程宏 crate 几乎总是链接到编译器提供的 [`proc_macro` crate]。`proc_macro` crate 提供编写过程宏所需的类型和工具来让编写更容易。

这个 crate 主要包含一个 [`TokenStream`] 类型。过程宏在*标记流(token streams)*上操作，而不是在 AST 节点上操作，对于编译器和过程宏来说，这是一个随着时间推移更加稳定的接口。*标记流*大致相当于 `Vec<TokenTree>`，其中 `TokenTree` 可以大致视为词法标记码。例如，`foo` 是 `Ident` 标记码， `.` 是 `Punct` 标记码， `1.2` 是一个 `Literal` 标记码。不同于 `Vec<TokenTree>`，`TokenStream` 的克隆成本很低。

所有标记码都有一个关联的 `Span`。 `Span` 是一个不透明的值，不能被修改，但可以被制造。 `Span` 表示程序内的源代码范围，主要用于错误报告。您可以修改任何标记码的 `Span`。

### 过程宏的卫生性

过程宏是*不卫生的*。这意味着它们表现地好像输出的标记流被简单地内联编写到它后面的代码中一样。这意味着它会受到外部数据项的影响，也会影响到外部的导入。

鉴于此限制，宏作者需要小心地确保他们的宏能在尽可能多的上下文中工作。这通常包括对库中数据项使用绝对路径(例如，使用 `::std::option::Option` 而不是 `Option`)，或者确保生成的函数具有不太可能与其他函数冲突的名称(如`__internal_foo`，而不是 `foo`)。

### 类函数过程宏

*类函数过程宏*是使用宏调用运算符（`!`）调用的过程宏。

这些宏是由一个带有 `proc_macro` [属性]和 `(TokenStream) -> TokenStream` 签名的 [公有]可见性[函数]。输入 [`TokenStream`] 是宏调用的定界符内的内容，输出 [`TokenStream`] 将替换整个宏调用。

例如，下面的宏定义忽略它的输入，并将函数 `answer` 输出到它的作用域。

<!-- ignore: test doesn't support proc-macro -->
```rust,ignore
# #![crate_type = "proc-macro"]
extern crate proc_macro;
use proc_macro::TokenStream;

#[proc_macro]
pub fn make_answer(_item: TokenStream) -> TokenStream {
    "fn answer() -> u32 { 42 }".parse().unwrap()
}
```

然后我们用它在一个二进制 crate 里打印 “42” 到标准输出。

<!-- ignore: requires external crates -->
```rust,ignore
extern crate proc_macro_examples;
use proc_macro_examples::make_answer;

make_answer!();

fn main() {
    println!("{}", answer());
}
```

类函数的过程宏可以在任何宏调用位置调用，包括[语句]、[表达式]、[模式]、[类型表达式]、[数据项]可以出现的位置（包括[`extern` 块]里、固有[实现]和 trait[实现]里、以及 [trait 定义]里）。

### 派生宏

*派生宏*为[`derive` 属性]定义新的输入。给定[结构体]、[枚举]或[联合体]的标记流，派生宏就可以创建新的[数据项]。它们还可以定义[派生宏辅助属性]。

自定义派生宏由带有 `proc_macro_derive` 属性和 `(TokenStream) -> TokenStream` 签名的[公有]可见性[函数]定义。

输入 [`TokenStream`] 是带有 `derive` 属性的数据项的标记流。输出 [`TokenStream`] 必须是一组数据项，然后将这组数据项追加到输入 [`TokenStream`] 中的那条数据项所在的[模块]或[块]中。

下面是派生宏的一个示例。它没有对输入执行任何有用的操作，只是附加了一个函数 `answer`。

<!-- ignore: test doesn't support proc-macro -->
```rust,ignore
# #![crate_type = "proc-macro"]
extern crate proc_macro;
use proc_macro::TokenStream;

#[proc_macro_derive(AnswerFn)]
pub fn derive_answer_fn(_item: TokenStream) -> TokenStream {
    "fn answer() -> u32 { 42 }".parse().unwrap()
}
```

然后使用这个派生宏：

<!-- ignore: requires external crates -->
```rust,ignore
extern crate proc_macro_examples;
use proc_macro_examples::AnswerFn;

#[derive(AnswerFn)]
struct Struct;

fn main() {
    assert_eq!(42, answer());
}
```

#### 派生宏辅助属性

派生宏可以将额外的[属性]添加到它们所在的[数据项]的作用域中。这所述属性称为*派生宏辅助属性*。这些属性是[惰性的]，它们的唯一目的是输入到定义它们的派生宏中。也就是说，所有宏都可以看到它们。

定义辅助属性的方法是在 `proc_macro_derive` 宏中放置一个 `attributes` 键，并使用逗号分隔的标识符列表，这些标识符是辅助属性的名称。

例如，下面的派生宏定义了一个辅助属性 `helper`，但最终没有对它做任何事情。

<!-- ignore: test doesn't support proc-macro -->
```rust,ignore
# #![crate_type="proc-macro"]
# extern crate proc_macro;
# use proc_macro::TokenStream;

#[proc_macro_derive(HelperAttr, attributes(helper))]
pub fn derive_helper_attr(_item: TokenStream) -> TokenStream {
    TokenStream::new()
}
```

然后在一个结构体上使用这个派生宏：

<!-- ignore: requires external crates -->
```rust,ignore
#[derive(HelperAttr)]
struct Struct {
    #[helper] field: ()
}
```

### 属性宏

*属性宏*定义可以附加到[数据项]上的新的[外部属性]，这些数据项包括 [`extern` 块]、固有[实现]、trate [实现]， 以及[trait 定义]中的数据项。

属性宏由带有 `proc_macro_attribute` 属性和 `(TokenStream, TokenStream) -> TokenStream` 签名的[公有]可见性[函数]定义。签名中的第一个 [`TokenStream`] 是属性名称后面带定界符的标记树，不包括外部定界符。如果该属性作为裸属性给出，则第一个 [`TokenStream`] 值为空。第二个 [`TokenStream`] 是数据项的其余部分，包括该数据项的其他属性。返回的 [`TokenStream`] 将项替换为任意数量的数据项。

例如，下面这个属性宏接受输入流并按原样返回，实际上对属性并无操作。

<!-- ignore: test doesn't support proc-macro -->
```rust,ignore
# #![crate_type = "proc-macro"]
# extern crate proc_macro;
# use proc_macro::TokenStream;

#[proc_macro_attribute]
pub fn return_as_is(_attr: TokenStream, item: TokenStream) -> TokenStream {
    item
}
```

下面的示例显示了属性宏看到的字符串化的 [`TokenStream`]。输出将显示在编译时的编译器输出窗口中。（具体的以 "out:"为前缀的）输出内容也都在下面每个示例函数后面的注释中给出了。

<!-- ignore: test doesn't support proc-macro -->
```rust,ignore
// my-macro/src/lib.rs
# extern crate proc_macro;
# use proc_macro::TokenStream;

#[proc_macro_attribute]
pub fn show_streams(attr: TokenStream, item: TokenStream) -> TokenStream {
    println!("attr: \"{}\"", attr.to_string());
    println!("item: \"{}\"", item.to_string());
    item
}
```

<!-- ignore: requires external crates -->
```rust,ignore
// src/lib.rs
extern crate my_macro;

use my_macro::show_streams;

// Example: Basic function
#[show_streams]
fn invoke1() {}
// out: attr: ""
// out: item: "fn invoke1() { }"

// Example: Attribute with input
#[show_streams(bar)]
fn invoke2() {}
// out: attr: "bar"
// out: item: "fn invoke2() {}"

// Example: Multiple tokens in the input
#[show_streams(multiple => tokens)]
fn invoke3() {}
// out: attr: "multiple => tokens"
// out: item: "fn invoke3() {}"

// Example:
#[show_streams { delimiters }]
fn invoke4() {}
// out: attr: "delimiters"
// out: item: "fn invoke4() {}"
```

[属性宏]: #属性宏
[Cargo的构建脚本]: ../cargo/reference/build-scripts.html
[派生宏]: #派生宏
[类函数宏]: #类函数宏
[`TokenStream`]: ../proc_macro/struct.TokenStream.html
[`TokenStream`s]: ../proc_macro/struct.TokenStream.html
[`compile_error`]: ../std/macro.compile_error.html
[`derive` attribute]: attributes/derive.md
[`extern` blocks]: items/external-blocks.md
[`macro_rules`]: macros-by-example.md
[`proc_macro` crate]: ../proc_macro/index.html
[属性]: attributes.md
[块]: expressions/block-expr.md
[crate 类型]: linkage.md
[派生宏辅助属性]: #派生宏辅助属性
[枚举]: items/enumerations.md
[表达式]: expressions.md
[函数]: items/functions.md
[实现]: items/implementations.md
[惰性的]: attributes.md#活动属性和惰性属性
[数据项]: items.md
[模块]: items/modules.md
[模式]: patterns.md
[公有]: visibility-and-privacy.md
[语句]: statements.md
[结构体]: items/structs.md
[trait 定义]: items/traits.md
[类型表达式]: types.md#类型表达式
[类型]: types.md
[联合体]: items/unions.md
