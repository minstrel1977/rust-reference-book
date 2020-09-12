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

`macro_rules`允许用户以声明方式定义语法扩展。我们称这种扩展为“声明宏（macros by example）”或简称“宏”。

每个宏都有一个名称和一个或多个*规则*。每个规则都有两个部分：一个*匹配器*，描述它匹配的语法；一个*转换器*，描述成功匹配后的将执行的替代调用语法。匹配器和转换器都必须由定界符包围。宏可以扩展到表达式、语句、数据项（包括 trait、impl 和外来数据项）、类型或模式。

## 转换

当调用宏时，宏扩展程序按名称查找宏调用，并依次尝试每个宏规则。宏会根据第一个成功的匹配进行转换；即使转换结果导致错误，也不会尝试后续的匹配。在匹配时，不执行预判；如果编译器不能明确地确定如何一次解析一个标记码的宏调用，则会导致错误。在下面的示例中，编译器不会提前查看标识符，以查看后跟的标记码是否为 ')'，即使这样可以明确地解析调用：

```rust,compile_fail
macro_rules! ambiguity {
    ($($i:ident)* $j:ident) => { };
}

ambiguity!(error); // 错误: 局部歧义
```

在匹配器和转换器中，标记码 `$` 用于从宏引擎中调用特殊行为(下文[元变量]和[重复]中有详述)。不属于此类调用的标记码将按字面意义进行匹配和转录，只有一个例外。例外情况是匹配器的外部定界符将匹配任何一对定界符。因此，比如匹配器 `(())` 将匹配'{()}'，而不是 `{{}}`。字符 `$` 不能按字面意义匹配或转换。

当将匹配片段转发给另一个声明宏时，第二个宏中的匹配器看到的将是此匹配片段类型的不透明<abbr title="AST：Abstract Syntax Tree">抽象语法树</abbr>。第二个宏不能使用<abbr title="literal tokens">字面量标记码</abbr>来匹配匹配器中的片段，只能使用相同类型的<abbr title="fragment specifier">片段分类符</abbr>。`ident`, `lifetime`, 和 `tt` 片段类型是一个例外，*可以*与字面量标记码进行匹配。以下代码说明了这一限制：

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

在匹配器中，`$`*名称*`:`*片段分类符* 匹配指定类型的 Rust 语法片段，并将其绑定到元变量`$`*名称*上。有效的片段分类符是：

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

在转换器中，元变量仅被简单地用`$`*名称* 这种形式引用（因为片段类型是在匹配器中指定的）。元变量最终将被替换为匹配它们的语法元素。元变量关键字 `$crate` 可以用来引用当前的 crate（请参阅下面的[卫生性]章节）。元变量可以被多次转换，也可以完全不转换。

## 重复

在匹配器和转换器中，通过将需要重复的标记码放在 `$(`…`)` 内来代表重复，后跟一个重复运算符，这两者之间可以防止一个可选的分隔标记码。分隔标记码可以是除定界符或某个重复运算符之外的任何记号，分号 `;` 和逗号 `,` 是最常见的。例如： `$( $i:ident ),*` 表示用逗号分隔的任何数量的标识符，且允许嵌套式重复。 

重复运算符为：

- `*` — 表示重复任意次数。
- `+` — 表示至少重复一次.
- `?` — 表示一个可选片段，可以出现零次或一次.

因为 `?` 表示最多出现一次，所以它不能与分隔标记码一起使用。

重复片段匹配并转换为指定数量的片段，由分隔标记码分隔。元变量与它们对应的片段的每次重复相匹配。例如，上面的 `$( $i:ident ),*` 示例将 `$i` 匹配到（标识符）列表中的所有标识符。

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

其次，它可以用于从另一个 crate 里导入宏，方法是将它附加到出现在 crate 根模块中的 `extern crate` 声明前。以这种方式导入的宏被导入到当前 crate 的 prelude 里，而不是文本导入，这意味着它们可以被任何其他名称遮蔽。虽然可以在导入语句之前使用 `#[macro_use]` 导入宏，但如果发生冲突，则最后导入的宏将获胜。可选地，可以使用 [_MetaListIdents_] 语法指定要导入的宏列表（如果将 `#[macro_use]` 应用于模块，则不支持此操作）。 

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

> **Version & Edition Differences**: Prior to Rust 1.30, `$crate` and
> `local_inner_macros` (below) were unsupported. They were added alongside
> path-based imports of macros (described above), to ensure that helper macros
> did not need to be manually imported by users of a macro-exporting crate.
> Crates written for earlier versions of Rust that use helper macros need to be
> modified to use `$crate` or `local_inner_macros` to work well with path-based
> imports.

When a macro is exported, the `#[macro_export]` attribute can have the
`local_inner_macros` keyword added to automatically prefix all contained macro
invocations with `$crate::`. This is intended primarily as a tool to migrate
code written before `$crate` was added to the language to work with Rust 2018's
path-based imports of macros. Its use is discouraged in new code.

```rust
#[macro_export(local_inner_macros)]
macro_rules! helped {
    () => { helper!() } // Automatically converted to $crate::helper!().
}

#[macro_export]
macro_rules! helper {
    () => { () }
}
```

## Follow-set Ambiguity Restrictions

The parser used by the macro system is reasonably powerful, but it is limited in
order to prevent ambiguity in current or future versions of the language. In
particular, in addition to the rule about ambiguous expansions, a nonterminal
matched by a metavariable must be followed by a token which has been decided can
be safely used after that kind of match.

As an example, a macro matcher like `$i:expr [ , ]` could in theory be accepted
in Rust today, since `[,]` cannot be part of a legal expression and therefore
the parse would always be unambiguous. However, because `[` can start trailing
expressions, `[` is not a character which can safely be ruled out as coming
after an expression. If `[,]` were accepted in a later version of Rust, this
matcher would become ambiguous or would misparse, breaking working code.
Matchers like `$i:expr,` or `$i:expr;` would be legal, however, because `,` and
`;` are legal expression separators. The specific rules are:

  * `expr` and `stmt` may only be followed by one of: `=>`, `,`, or `;`.
  * `pat` may only be followed by one of: `=>`, `,`, `=`, `|`, `if`, or `in`.
  * `path` and `ty` may only be followed by one of: `=>`, `,`, `=`, `|`, `;`,
    `:`, `>`, `>>`, `[`, `{`, `as`, `where`, or a macro variable of `block`
    fragment specifier.
  * `vis` may only be followed by one of: `,`, an identifier other than a
    non-raw `priv`, any token that can begin a type, or a metavariable with a
    `ident`, `ty`, or `path` fragment specifier.
  * All other fragment specifiers have no restrictions.

When repetitions are involved, then the rules apply to every possible number of
expansions, taking separators into account. This means:

  * If the repetition includes a separator, that separator must be able to
    follow the contents of the repetition.
  * If the repetition can repeat multiple times (`*` or `+`), then the contents
    must be able to follow themselves.
  * The contents of the repetition must be able to follow whatever comes
    before, and whatever comes after must be able to follow the contents of the
    repetition.
  * If the repetition can match zero times (`*` or `?`), then whatever comes
    after must be able to follow whatever comes before.


For more detail, see the [formal specification].

[Hygiene]: #hygiene
[IDENTIFIER]: identifiers.md
[标识符或关键字]: identifiers.md
[生命周期标记码]: tokens.md#lifetimes-and-loop-labels
[Metavariables]: #metavariables
[Repetitions]: #repetitions
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
[formal specification]: macro-ambiguity.md
[token]: tokens.md
