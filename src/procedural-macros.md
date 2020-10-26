## Procedural Macros
## 过程宏

>[procedural-macros.md](https://github.com/rust-lang/reference/blob/master/src/procedural-macros.md)\
>commit: a1ef5a09c0281b0f2a65c18670e927ead61eb1b2 \
>本译文最后维护日期：2020-10-16

*过程宏*允许在执行函数时创建句法扩展。过程宏有三种形式:

* [类函数宏(function-like macros)][Function-like macros] - `custom!(...)`
* [派生宏(derive macros)][Derive macros]- `#[derive(CustomDerive)]`
* [属性宏(attribute macros)][Attribute macros] - `#[CustomAttribute]`

过程宏允许您在编译时运行对 Rust 句法进行操作的代码，同时消费并生成 Rust 句法。您可以将过程宏看作是从一个 <abbr title="抽象句法树：Abstract Syntax Tree">AST</abbr> 到另一个 <abbr title="抽象句法树：Abstract Syntax Tree">AST</abbr> 的函数。

过程宏必须在 [crate 类型][crate type]为 `proc-macro` 的 crate 中定义。

> **注意**: 使用 Cargo 时，定义过程宏的 crate 的配置文件里要使用 `proc-macro`键做如下设置：
>
> ```toml
> [lib]
> proc-macro = true
> ```

作为函数，它们要么有返回句法，要么 panic，要么永无休止地循环。返回句法根据过程宏的类型替换或添加句法；panic 会被编译器捕获，并将其转化为编译器错误；无限循，不会被编译器不会捕获，但会导致编译器被挂起。

过程宏在编译期运行，因此具有与编译器相同的资源。例如，它可以访问的标准输入、标准错误输出和标准输出与编译器可以访问的相同。类似地，文件访问也是一样的。因此，过程宏与 [Cargo 的构建脚本][Cargo's build scripts] 具有相同的安全考量。

过程宏有两种报告错误的方法。首先是 panic；第二个是发布 [`compile_error`] 性质的宏调用。

### The `proc_macro` crate

过程宏 crate 几乎总是会去链接编译器提供的 [`proc_macro` crate]。`proc_macro` crate 提供编写过程宏所需的类型和工具来让编写更容易。

这个 crate 主要包含一个 [`TokenStream`] 类型。过程宏在*标记流(token streams)*上操作，而不是在 AST 节点上操作，因为这对于编译器和过程宏的构建目标来说，这是一个随着时间推移要稳定得多的接口。*标记流*大致相当于 `Vec<TokenTree>`，其中 `TokenTree` 可以大致视为词法标记码。例如，`foo` 是 `Ident` 标记码， `.` 是 `Punct` 标记码， `1.2` 是一个 `Literal` 标记码。不同于 `Vec<TokenTree>`，`TokenStream` 的克隆成本很低。

所有标记码都有一个关联的 `Span`。 `Span` 是一个不透明的值，不能被修改，但可以被制造。 `Span` 表示程序内的源代码范围，主要用于错误报告。可以修改任何标记码的 `Span`。

### Procedural macro hygiene
### 过程宏的卫生性

过程宏是*非卫生的(unhygienic)*。这意味着它的行为就好像它的输出令牌流是直接内联写入它前后的代码中一样。这意味着它会受到外部数据项的影响，也会影响外部导入。

鉴于此限制，宏作者需要小心地确保他们的宏能在尽可能多的上下文中正常工作。这通常包括对库中数据项使用绝对路径(例如，使用 `::std::option::Option` 而不是 `Option`)，或者确保生成的函数具有不太可能与其他函数冲突的名称(如 `__internal_foo`，而不是 `foo`)。

### Function-like procedural macros
### 类函数过程宏

*类函数过程宏*是使用宏调用运算符（`!`）调用的过程宏。

这种宏是由一个带有 `proc_macro` [属性][attribute]和 `(TokenStream) -> TokenStream` 签名的 [公有][public]可见性[函数][function]定义。输入 [`TokenStream`] 是由宏调用的定界符界定的内容，输出 [`TokenStream`] 将替换整个宏调用。

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

类函数过程宏可以在任何宏调用位置调用，这些位置包括[语句][statements]、[表达式][expressions]、[模式][patterns]、[类型表达式][type expressions]、[数据项][item]可以出现的位置（包括[`extern`块][`extern` blocks]里、固有[实现][implementations]和 trait实现里、以及 [trait定义][trait definitions]里）。

### Derive macros
### 派生宏

*派生宏*为[派生(`derive`)属性][`derive` attribute]定义新输入。这类宏在给定[结构体(`struct`)][struct]、[枚举(`enum`)][enum]或[联合体(`union`)][union]标记流的情况下创建新[数据项][items]。它们也可以定义[派生宏辅助属性][derive macro helper attributes]。

自定义派生宏由带有 `proc_macro_derive` 属性和 `(TokenStream) -> TokenStream` 签名的[公有][public]可见性[函数][function]定义。

输入 [`TokenStream`] 是带有 `derive` 属性的数据项的标记流。输出 [`TokenStream`] 必须是一组数据项，然后将这组数据项追加到输入 [`TokenStream`] 中的那条数据项所在的[模块][module]或[块][block]中。

下面是派生宏的一个示例。它没有对输入执行任何有用的操作，只是追加了一个函数 `answer`。

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

#### Derive macro helper attributes
#### 派生宏辅助属性

派生宏可以将额外的[属性][attributes]添加到它们所在的[数据项][item]的作用域中。这些属性被称为*派生宏辅助属性*。这些属性是[惰性的][inert]，它们存在的唯一目的是将这些属性在使用现场获得的属性值反向输入到定义它们的派生宏中。也就是说所有该宏的宏应用都可以看到它们。

定义辅助属性的方法是在 `proc_macro_derive` 宏中放置一个 `attributes` 键，此键带有一个使用逗号分隔的标识符列表，这些标识符是辅助属性的名称。

例如，下面的派生宏定义了一个辅助属性 `helper`，但最终没有用它做任何事情。

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

### Attribute macros
### 属性宏

属性宏定义了可以附加到项上的新的外部属性，包括extern块、固有和trait实现以及trait定义中的项。
*属性宏*定义可以附加到[数据项][items]上的新的[外部属性][attributes]，这些数据项包括[外部(`extern`)块][`extern` blocks]、固有[实现][implementations]、trate实现，以及 [trait定义][trait definitions]中的数据项。

属性宏由带有 `proc_macro_attribute`[属性][attribute]和 `(TokenStream, TokenStream) -> TokenStream` 签名的[公有][public]可见性[函数][function]定义。签名中的第一个 [`TokenStream`] 是属性名称后面的定界标记树(delimited token tree)（不包括外层定界符）。如果该属性作为裸属性(bare attribute)给出，则第一个 [`TokenStream`] 值为空。第二个 [`TokenStream`] 是[数据项][item]的其余部分，包括该数据项的其他[属性][attributes]。返回的 [`TokenStream`] 将此属性宏应用的[数据项][item]替换为任意数量的数据项。

例如，下面这个属性宏接受输入流并按原样返回，实际上对属性并空操作。

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

下面示例显示了属性宏看到的字符串化的 [`TokenStream`]。输出将显示在编译时的编译器输出窗口中。（具体格式是以 "out:"为前缀的）输出内容也都在后面每个示例函数后面的注释中给出了。

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

[Attribute macros]: #attribute-macros
[Cargo's build scripts]: ../cargo/reference/build-scripts.html
[Derive macros]: #derive-macros
[Function-like macros]: #function-like-procedural-macros
[`TokenStream`]: ../proc_macro/struct.TokenStream.html
[`TokenStream`s]: ../proc_macro/struct.TokenStream.html
[`compile_error`]: ../std/macro.compile_error.html
[`derive` attribute]: attributes/derive.md
[`extern` blocks]: items/external-blocks.md
[`macro_rules`]: macros-by-example.md
[`proc_macro` crate]: ../proc_macro/index.html
[attribute]: attributes.md
[attributes]: attributes.md
[block]: expressions/block-expr.md
[crate type]: linkage.md
[derive macro helper attributes]: #derive-macro-helper-attributes
[enum]: items/enumerations.md
[expressions]: expressions.md
[function]: items/functions.md
[implementations]: items/implementations.md
[inert]: attributes.md#active-and-inert-attributes
[item]: items.md
[items]: items.md
[module]: items/modules.md
[patterns]: patterns.md
[public]: visibility-and-privacy.md
[statements]: statements.md
[struct]: items/structs.md
[trait definitions]: items/traits.md
[type expressions]: types.md#type-expressions
[type]: types.md
[union]: items/unions.md

<!-- 2020-10-25 -->
<!-- checked -->