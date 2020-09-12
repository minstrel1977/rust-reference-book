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
use lazy_static::lazy_static; // Path-based import.

macro_rules! lazy_static { // Textual definition.
    (lazy) => {};
}

lazy_static!{lazy} // Textual lookup finds our macro first.
self::lazy_static!{} // Path-based lookup ignores our macro, finds imported one.
```

### Textual Scope

Textual scope is based largely on the order that things appear in source files,
and works similarly to the scope of local variables declared with `let` except
it also applies at the module level. When `macro_rules!` is used to define a
macro, the macro enters the scope after the definition (note that it can still
be used recursively, since names are looked up from the invocation site), up
until its surrounding scope, typically a module, is closed. This can enter child
modules and even span across multiple files:

<!-- ignore: requires external modules -->
```rust,ignore
//// src/lib.rs
mod has_macro {
    // m!{} // Error: m is not in scope.

    macro_rules! m {
        () => {};
    }
    m!{} // OK: appears after declaration of m.

    mod uses_macro;
}

// m!{} // Error: m is not in scope.

//// src/has_macro/uses_macro.rs

m!{} // OK: appears after declaration of m in src/lib.rs
```

It is not an error to define a macro multiple times; the most recent declaration
will shadow the previous one unless it has gone out of scope.

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
    // m!(1); // Error: no rule matches '1'
    m!(2);

    macro_rules! m {
        (3) => {};
    }
    m!(3);
}

m!(1);
```

Macros can be declared and used locally inside functions as well, and work
similarly:

```rust
fn foo() {
    // m!(); // Error: m is not in scope.
    macro_rules! m {
        () => {};
    }
    m!();
}


// m!(); // Error: m is not in scope.
```

### The `macro_use` attribute

The *`macro_use` attribute* has two purposes. First, it can be used to make a
module's macro scope not end when the module is closed, by applying it to a
module:

```rust
#[macro_use]
mod inner {
    macro_rules! m {
        () => {};
    }
}

m!();
```

Second, it can be used to import macros from another crate, by attaching it to
an `extern crate` declaration appearing in the crate's root module. Macros
imported this way are imported into the prelude of the crate, not textually,
which means that they can be shadowed by any other name. While macros imported
by `#[macro_use]` can be used before the import statement, in case of a
conflict, the last macro imported wins. Optionally, a list of macros to import
can be specified using the [_MetaListIdents_] syntax; this is not supported
when `#[macro_use]` is applied to a module.

<!-- ignore: requires external crates -->
```rust,ignore
#[macro_use(lazy_static)] // Or #[macro_use] to import all macros.
extern crate lazy_static;

lazy_static!{}
// self::lazy_static!{} // Error: lazy_static is not defined in `self`
```

Macros to be imported with `#[macro_use]` must be exported with
`#[macro_export]`, which is described below.

### Path-Based Scope

By default, a macro has no path-based scope. However, if it has the
`#[macro_export]` attribute, then it is declared in the crate root scope and can
be referred to normally as such:

```rust
self::m!();
m!(); // OK: Path-based lookup finds m in the current module.

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

Macros labeled with `#[macro_export]` are always `pub` and can be referred to
by other crates, either by path or by `#[macro_use]` as described above.

## Hygiene

By default, all identifiers referred to in a macro are expanded as-is, and are
looked up at the macro's invocation site. This can lead to issues if a macro
refers to an item or macro which isn't in scope at the invocation site. To
alleviate this, the `$crate` metavariable can be used at the start of a path to
force lookup to occur inside the crate defining the macro.

<!-- ignore: requires external crates -->
```rust,ignore
//// Definitions in the `helper_macro` crate.
#[macro_export]
macro_rules! helped {
    // () => { helper!() } // This might lead to an error due to 'helper' not being in scope.
    () => { $crate::helper!() }
}

#[macro_export]
macro_rules! helper {
    () => { () }
}

//// Usage in another crate.
// Note that `helper_macro::helper` is not imported!
use helper_macro::helped;

fn unit() {
    helped!();
}
```

Note that, because `$crate` refers to the current crate, it must be used with a
fully qualified module path when referring to non-macro items:

```rust
pub mod inner {
    #[macro_export]
    macro_rules! call_foo {
        () => { $crate::inner::foo() };
    }

    pub fn foo() {}
}
```

Additionally, even though `$crate` allows a macro to refer to items within its
own crate when expanding, its use has no effect on visibility. An item or macro
referred to must still be visible from the invocation site. In the following
example, any attempt to invoke `call_foo!()` from outside its crate will fail
because `foo()` is not public.

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
