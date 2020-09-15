# 声明宏

>[macros-by-example.md](https://github.com/rust-lang/reference/blob/master/src/macros-by-example.md)\
>commit 30ccf092c7c1d92b2398e28d4abf8c5ba6c31cba

> **<sup>句法</sup>**\
> _MacroRulesDefinition_ :\
> &nbsp;&nbsp; `macro_rules` `!` [IDENTIFIER] _MacroRulesDef_
>
> _MacroRulesDef_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `(` _MacroRules_ `)` `;`\
> &nbsp;&nbsp; | `[` _MacroRules_ `]` `;`\
> &nbsp;&nbsp; | `{` _MacroRules_ `}`
>
> _MacroRules_ :\
> &nbsp;&nbsp; _MacroRule_ ( `;` _MacroRule_ )<sup>\*</sup> `;`<sup>?</sup>
>
> _MacroRule_ :\
> &nbsp;&nbsp; _MacroMatcher_ `=>` _MacroTranscriber_
>
> _MacroMatcher_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `(` _MacroMatch_<sup>\*</sup> `)`\
> &nbsp;&nbsp; | `[` _MacroMatch_<sup>\*</sup> `]`\
> &nbsp;&nbsp; | `{` _MacroMatch_<sup>\*</sup> `}`
>
> _MacroMatch_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_Token_]<sub>_排除 $ 和 定界符_</sub>\
> &nbsp;&nbsp; | _MacroMatcher_\
> &nbsp;&nbsp; | `$` [IDENTIFIER] `:` _MacroFragSpec_\
> &nbsp;&nbsp; | `$` `(` _MacroMatch_<sup>+</sup> `)` _MacroRepSep_<sup>?</sup> _MacroRepOp_
>
> _MacroFragSpec_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `block` | `expr` | `ident` | `item` | `lifetime` | `literal`\
> &nbsp;&nbsp; | `meta` | `pat` | `path` | `stmt` | `tt` | `ty` | `vis`
>
> _MacroRepSep_ :\
> &nbsp;&nbsp; [_Token_]<sub>_排除 定界符 和 重复操作符_</sub>
>
> _MacroRepOp_ :\
> &nbsp;&nbsp; `*` | `+` | `?`
>
> _MacroTranscriber_ :\
> &nbsp;&nbsp; [_DelimTokenTree_]

`macro_rules`允许用户以声明方式定义句法扩展。我们称这种扩展为“声明宏（macros by example）”或简称“宏”。

每个宏都有一个名称和一个或多个*规则*。每个规则都有两个部分：一个*匹配器*，描述它匹配的句法；一个*转换器*，描述成功匹配后的将执行的替代调用句法。匹配器和转换器都必须由定界符包围。宏可以扩展到表达式、语句、数据项（包括 trait、impl 和外来数据项）、类型或模式。

## 转换

当调用宏时，宏扩展程序按名称查找宏调用，并依次尝试每个宏规则。宏会根据第一个成功的匹配进行转换；即使转换结果导致错误，也不会尝试后续的匹配。在匹配时，不执行预判；如果编译器不能明确地确定如何一次解析一个标记码的宏调用，则会导致错误。在下面的示例中，编译器不会提前查看标识符，以查看后跟的标记码是否为 ')'，即使这样可以明确地解析调用：

```rust,compile_fail
macro_rules! ambiguity {
    ($($i:ident)* $j:ident) => { };
}

ambiguity!(error); // 错误: 局部歧义
```

在匹配器和转换器中，标记码 `$` 用于从宏引擎中调用特殊行为(下文[元变量]和[重复]中有详述)。不属于此类调用的标记码将按字面意义进行匹配和转录，只有一个例外。例外情况是匹配器的外部定界符将匹配任何一对定界符。因此，比如匹配器 `(())` 将匹配'{()}'，而不是 `{{}}`。字符 `$` 不能按字面意义匹配或转换。

当将匹配片段转发给另一个声明宏时，第二个宏中的匹配器看到的将是此匹配片段类型的不透明<abbr title="AST：Abstract Syntax Tree">抽象句法树</abbr>。第二个宏不能使用<abbr title="literal tokens">字面量标记码</abbr>来匹配匹配器中的片段，只能使用相同类型的<abbr title="fragment specifier">片段分类符</abbr>。`ident`, `lifetime`, 和 `tt` 片段类型是一个例外，*可以*与字面量标记码进行匹配。以下代码说明了这一限制：

```rust,compile_fail
macro_rules! foo {
    ($l:expr) => { bar!($l); }
// ERROR:               ^^ no rules expected this token in macro call
}

macro_rules! bar {
    (3) => {}
}

foo!(3);
```

以下代码说明了如何在匹配 `tt` 片段之后直接匹配标记码：

```rust
// 成功编译
macro_rules! foo {
    ($l:tt) => { bar!($l); }
}

macro_rules! bar {
    (3) => {}
}

foo!(3);
```

## 元变量

在匹配器中，`$`*名称*`:`*片段分类符* 匹配指定类型的 Rust 句法片段，并将其绑定到元变量`$`*名称*上。有效的片段分类符是：

  * `item`: 一个 [_数据项_]
  * `block`: 一个 [_块表达式_]
  * `stmt`: 一条句尾没有分号的[_语句_]（需要分号的数据项语句除外）
  * `pat`: 一个 [_模式_]
  * `expr`: 一个 [_表达式_]
  * `ty`: 一个 [_类型_]
  * `ident`: 一个 [标识符或关键字]
  * `path`: 一条 [_类型路径_] 风格的路径
  * `tt`: 一个 [_标记树_]&nbsp;(单个[标记码]或匹配型定界符（matching delimiters）`（）`、`[]`、`{}` 中的标记)
  * `meta`: an [_属性值_], 属性的内容
  * `lifetime`: 一个 [生命周期标记码]
  * `vis`: 可能为空的[_可见性_]限定符
  * `literal`: 匹配 `-`<sup>?</sup>[_字面量表达式_]

在转换器中，元变量仅被简单地用`$`*名称* 这种形式引用（因为片段类型是在匹配器中指定的）。元变量最终将被替换为匹配它们的句法元素。元变量关键字 `$crate` 可以用来引用当前的 crate（请参阅下面的[卫生性]章节）。元变量可以被多次转换，也可以完全不转换。

## 重复

在匹配器和转换器中，通过将需要重复的标记码放在 `$(`…`)` 内来代表重复，后跟一个重复运算符，这两者之间可以防止一个可选的分隔符标记码()。分隔符标记码可以是除定界符或某个重复运算符之外的任何记号，分号 `;` 和逗号 `,` 是最常见的。例如： `$( $i:ident ),*` 表示用逗号分隔的任何数量的标识符，且允许嵌套式重复。 

重复运算符为：

- `*` — 表示重复任意次数。
- `+` — 表示至少重复一次.
- `?` — 表示一个可选片段，可以出现零次或一次.

因为 `?` 表示最多出现一次，所以它不能与分隔符一起使用。

重复片段匹配并转换为指定数量的片段，由分隔符标记码分隔。元变量与它们对应的片段的每次重复相匹配。例如，上面的 `$( $i:ident ),*` 示例将 `$i` 匹配到（标识符）列表中的所有标识符。

在转换过程中，重复会受到额外的限制，以便于编译器知道该如何正确地扩展它们：

1.  在转换器中，元变量必须与它在匹配器中出现的重复次数、种类和嵌套顺序完全相同。因此，对于匹配器 `$( $i:ident ),*`，转换器 `=> { $i }`, `=> { $( $( $i)* )* }` 和 `=> { $( $i )+ }` 都是非法的，但是 `=> { $( $i );* }` 是正确的，并用分号分隔的列表替换了逗号分隔的标识符列表。
2.  转换器中的每个重复必须至少包含一个元变量，以便确定扩展多少次。如果在同一个重复中出现多个元变量，则它们必须绑定到相同数量的片段上。例如，`( $( $i:ident ),* ; $( $j:ident ),* ) =>( $( ($i,$j) ),*` 必须绑定与 `$j` 片段相同数量的 `$i` 片段上。这意味着用 `(a, b, c; d, e, f`) 调用前面的宏是合法的，并且可扩展到 `((a,d), (b,e), (c,f))`，但是 `(a, b, c; d, e)` 是非法的，因为其数量不同。此要求适用于嵌套重复的每一层。

## 作用域，导出，以及导入

由于历史原因，声明宏的作用域并不完全像数据项那样工作。宏有两种形式的作用域：文本作用域和基于路径的作用域。文本作用域基于代码在源文件中出现的顺序，甚至是跨多个文件出现的顺序，并且是默认的作用域。下面将进一步解释这个。基于路径的作用域与数据项作用域的工作方式完全相同。宏的范围、导出和导入主要由属性控制。

当声明宏被非限定标识符（不是多重路径的一部分）调用时，首先在文本作用域中查找。如果文本作用域中没有任何结果，则继续在基于路径的作用域中查找。如果宏的名称由路径限定，则只在基于路径的作用域中查找。

<!-- ignore: requires external crates -->
```rust,ignore
use lazy_static::lazy_static; // 基于路径的导入.

macro_rules! lazy_static { // 文本定义.
    (lazy) => {};
}

lazy_static!{lazy} // 首先通过文本作用域来查找我们的宏.
self::lazy_static!{} // 忽略文本作用域查找，直接使用基于路径的查找方式找到一个导入的宏.
```

### 文本作用域

文本作用域很大程度上取决于事物在源文件中的出现顺序，其工作方式与用 `let` 声明的局部变量的作用域类似，只不过它也适用于模块级别。当使用 `macro_rules!` 定义宏时，宏在定义之后进入其作用域（请注意，它可以递归使用，因为名称是从调用位置查找的），直到其周围的作用域（通常是模块）关闭为止。文本作用域可以进入子模块，甚至跨越多个文件：

<!-- ignore: requires external modules -->
```rust,ignore
//// src/lib.rs
mod has_macro {
    // m!{} // 报错: m 未在作用域内.

    macro_rules! m {
        () => {};
    }
    m!{} // OK: 在声明 m 后使用.

    mod uses_macro;
}

// m!{} // Error: m 未在作用域内.

//// src/has_macro/uses_macro.rs

m!{} // OK: m 在上层模块文件 src/lib.rs 中声明后使用
```

多次定义宏并不报错；除非超出作用域，否则最近的宏声明将遮蔽前一个声明。

```rust
macro_rules! m {
    (1) => {};
}

m!(1);

mod inner {
    m!(1);

    macro_rules! m {
        (2) => {};
    }
    // m!(1); // 报错: 没有设定规则来匹配 '1'
    m!(2);

    macro_rules! m {
        (3) => {};
    }
    m!(3);
}

m!(1);
```

宏也可以在函数内部声明和使用，工作原理类似：

```rust
fn foo() {
    // m!(); // 报错: m 未在作用域内.
    macro_rules! m {
        () => {};
    }
    m!();
}


// m!(); // Error: m 未在作用域内.
```

### `macro_use` 属性

*`macro_use` 属性*有两种用途。首先，它可以通过作用于模块的方式让模块内的宏的作用域在模块关闭时不结束：

```rust
#[macro_use]
mod inner {
    macro_rules! m {
        () => {};
    }
}

m!();
```

其次，它可以用于从另一个 crate 里导入宏，方法是将它附加到出现在 crate 根模块中的 `extern crate` 声明前。以这种方式导入的宏被导入到当前 crate 的预导入包里，而不是文本导入，这意味着它们可以被任何其他名称遮蔽。虽然可以在导入语句之前使用 `#[macro_use]` 导入宏，但如果发生冲突，则最后导入的宏将获胜。可选地，可以使用 [_MetaListIdents_] 句法指定要导入的宏列表（如果将 `#[macro_use]` 应用于模块，则不支持此操作）。 

<!-- ignore: requires external crates -->
```rust,ignore
#[macro_use(lazy_static)] // 或者使用 #[macro_use] 来导入所有宏.
extern crate lazy_static;

lazy_static!{}
// self::lazy_static!{} // 报错: lazy_static 没在 `self` 中定义
```

要用 `#[macro_use]` 导入的宏必须使用 `#[macro_export]` 导出，下文会有讲解。

### 基于路径的作用域

默认情况下，宏没有基于路径的作用域。但是，如果它具有 `#[macro_export]` 属性，那么它会在当前 clate 的根作用域中被声明，并且通常可以这样引用：

```rust
self::m!();
m!(); // OK: 基于路径的查找发现 m 在当前模块中有声明.

mod inner {
    super::m!();
    crate::m!();
}

mod mac {
    #[macro_export]
    macro_rules! m {
        () => {};
    }
}
```

标有 `#[macro_export]` 的宏始终是 `pub` 的，可以通过路径或上面所述的 `#[macro_use]` 的方式让其他 crate 引用。

## 卫生性（Hygiene）

默认情况下，宏中引用的所有标识符都按原样展开，并在宏的调用位置上查找。如果宏引用的数据项或宏不在调用位置的作用域内，则这可能会导致问题。为了缓解这种情况，可以在路径的开头使用`$crate` 元变量，以强制在定义宏的 crate 中进行查找。

<!-- ignore: requires external crates -->
```rust,ignore
//// 在 `helper_macro` crate 中.
#[macro_export]
macro_rules! helped {
    // () => { helper!() } // 这可能会导致错误，因为 'helper' 在当前作用域之后才定义.
    () => { $crate::helper!() }
}

#[macro_export]
macro_rules! helper {
    () => { () }
}

//// 在另一个 crate 中使用.
// 注意 `helper_macro::helper` 没有导入!
use helper_macro::helped;

fn unit() {
    helped!();
}
```

请注意，由于 `$crate` 指的是当前的 crate，因此在引用非宏数据项时，它必须与完全限定的模块路径一起使用：

```rust
pub mod inner {
    #[macro_export]
    macro_rules! call_foo {
        () => { $crate::inner::foo() };
    }

    pub fn foo() {}
}
```

此外，尽管 `$crate` 允许宏在扩展时引用其自身 crate 中的数据项，但它的使用对可见性没有影响。引用的数据项或宏必须仍然在调用位置可见。在下面的示例中，任何试图从其 crate 外部调用 `call_foo!()` 的行为都将失败，因为 `foo()` 不是公有的。（译者注：下面的调用是可以，原文上没有给出 crate 外部调用的例子）

```rust
#[macro_export]
macro_rules! call_foo {
    () => { $crate::foo() };
}

fn foo() {}
```

> **版本/版次差异**：在 Rust 1.30 之前，`$crate` 和 `local_inner_macros` （以下）不受支持。该版本开始，它们与基于路径的宏导入（如上所述）一起添加，以确保辅助宏不需要由宏导出 crate 的用户手动导入。如果要让 Rust 的早期版本编写的 crate 使用辅助宏，需要修改为使用 `$crate` 或 `local_inner_macros`，以便与基于路径的导入一起工作。

当一个宏被导出时，`#[macro_export]` 属性里可以添加`local_inner_macros`关键字，可以自动为（该属性修饰的宏）内包含的所有宏调自动添加 `$crate::` 前缀。这主要是作为一个工具，用于迁移那些在引入 `$crate` 之前的版本编写的 Rust 代码，以便它们能与 Rust 2018 版中基于路径的宏导入一起工作。在使用新版本编写的代码中不鼓励使用它。

```rust
#[macro_export(local_inner_macros)]
macro_rules! helped {
    () => { helper!() } // 自动转换为 $crate::helper!().
}

#[macro_export]
macro_rules! helper {
    () => { () }
}
```

## 随集歧义限制（Follow-set Ambiguity Restrictions）

宏系统使用的解析器相当强大，但为了防止其与语言的当前或未来版本产生歧义，对它做出了限制。特别地，除了关于歧义性展开的规则外，由元变量匹配的非终结符必须后跟一个已确定可以在这种匹配之后安全使用的标记码。

例如，像 `$i:expr [ , ]` 这样的宏匹配器在现今的 Rust 中理论上是可以接受的，因为`[,]` 不能是合法表达式的一部分，因此解析始终是明确的。但是，由于`[`可以开始尾随表达式，`[`不是一个可以安全排除在表达式后面出现的字符。如果在接下来的 Rust 版本中接受了 `[,]`，那么这个匹配器就会产生歧义或是错误解析，破坏正常代码。但是，像`$i:expr,` 或 `$i:expr;` 这样的匹配符是合法的，因为 `,` 和`;` 是合法的表达式分隔符。具体规则是：

  * `expr` 和 `stmt` 或许只能后跟一个： `=>`, `,`, 或 `;`。
  * `pat` 或许只能后跟一个： `=>`, `,`, `=`, `|`, `if`, 或 `in`。
  * `path` 和 `ty` 或许只能后跟一个： `=>`, `,`, `=`, `|`, `;`, `:`, `>`, `>>`, `[`, `{`, `as`, `where`, 或 `block` 片段分类符的宏变量。
  * `vis` 或许只能后跟一个： `,`, 非原生 `priv`以外的标识符，可以开始类型的任何标记码， 或者带有任何 `ident`, `ty`, `path` 片段分类符的元变量。
  * 其它所有的片段分类符没有限制。

当涉及到重复时，这些规则适用于所有可能的展开次数，注意需将分隔符考虑在内。这意味着：

  * 如果重复包含分隔符，则分隔符必须能够跟随重复的内容。
  * 如果重复可以重复多次(`*` 或 `+`)，那么每一层重复里的内容必须形式统一。
  * 重复的内容必须能够和它前面的内容形式保持一致，后面的内容也必须遵循相同的形式。
  * 如果重复可以匹配零次(`*` 或 `?`)，那么后面的内容必须能够和前面的内容形式一致。

>（译者注：这几条规则直译很难理解，但是我又对这些规则理解不深刻，所以只能凑合这先这么意译，等以后理解深刻了再来修改，那先在这儿打个TobeModify的标记）。
有关更多详细信息，请参阅[正式规范]。

[卫生性]: #hygiene
[IDENTIFIER]: identifiers.md
[标识符或关键字]: identifiers.md
[生命周期标记码]: tokens.md#lifetimes-and-loop-labels
[元变量]: #metavariables
[重复]: #repetitions
[_属性值_]: attributes.md
[_块表达式_]: expressions/block-expr.md
[_DelimTokenTree_]: macros.md
[_表达式_]: expressions.md
[_数据项_]: items.md
[_字面量表达式_]: expressions/literal-expr.md
[_MetaListIdents_]: attributes.md#meta-item-attribute-syntax
[_模式_]: patterns.md
[_语句_]: statements.md
[_标记树_]: macros.md#macro-invocation
[_Token_]: tokens.md
[_类型路径_]: paths.md#类型路径
[_类型_]: types.md#type-expressions
[_可见性_]: visibility-and-privacy.md
[正式规范]: macro-ambiguity.md
[标记码]: tokens.md
