# 关键字

>[notation.md](https://github.com/rust-lang/reference/blob/master/src/keywords.md)\
>commit 64923185890763048d190ce92cb668b58acbc49a

Rust 将关键字分为三类：

  - [严格关键字](#严格关键字)
  - [保留关键字](#保留关键字)
  - [弱关键字](#弱关键字)

## 严格关键字

这类关键字只能在正确的上下文中使用。它们不能用作下列名称：

* [项目]
* [变量]和函数参数
* 字段和[枚举的变体]
* [类型参数]
* 生命周期参数或者[循环标签]
* [宏]或[属性]
* [宏占位符]
* [crate]

> **<sup>词法分析器:<sup>**\
> KW_AS             : `as`\
> KW_BREAK          : `break`\
> KW_CONST          : `const`\
> KW_CONTINUE       : `continue`\
> KW_CRATE          : `crate`\
> KW_ELSE           : `else`\
> KW_ENUM           : `enum`\
> KW_EXTERN         : `extern`\
> KW_FALSE          : `false`\
> KW_FN             : `fn`\
> KW_FOR            : `for`\
> KW_IF             : `if`\
> KW_IMPL           : `impl`\
> KW_IN             : `in`\
> KW_LET            : `let`\
> KW_LOOP           : `loop`\
> KW_MATCH          : `match`\
> KW_MOD            : `mod`\
> KW_MOVE           : `move`\
> KW_MUT            : `mut`\
> KW_PUB            : `pub`\
> KW_REF            : `ref`\
> KW_RETURN         : `return`\
> KW_SELFVALUE      : `self`\
> KW_SELFTYPE       : `Self`\
> KW_STATIC         : `static`\
> KW_STRUCT         : `struct`\
> KW_SUPER          : `super`\
> KW_TRAIT          : `trait`\
> KW_TRUE           : `true`\
> KW_TYPE           : `type`\
> KW_UNSAFE         : `unsafe`\
> KW_USE            : `use`\
> KW_WHERE          : `where`\
> KW_WHILE          : `while`

以下关键词从 2018 版开始启用。

> **<sup>词法分析器 2018+</sup>**\
> KW_ASYNC          : `async`\
> KW_AWAIT          : `await`\
> KW_DYN            : `dyn`

## 保留关键字

这类关键字还没有被使用，但是它们被保留以备将来使用。它们具有与严格关键字相同的限制。这样做的原因是通过禁止当前程序使用这些关键字，从而使当前程序向前兼容 Rust 的未来版本。

> **<sup>词法分析器</sup>**\
> KW_ABSTRACT       : `abstract`\
> KW_BECOME         : `become`\
> KW_BOX            : `box`\
> KW_DO             : `do`\
> KW_FINAL          : `final`\
> KW_MACRO          : `macro`\
> KW_OVERRIDE       : `override`\
> KW_PRIV           : `priv`\
> KW_TYPEOF         : `typeof`\
> KW_UNSIZED        : `unsized`\
> KW_VIRTUAL        : `virtual`\
> KW_YIELD          : `yield`

以下关键词从 2018 版开始保留。

> **<sup>词法分析器 2018+</sup>**\
> KW_TRY   : `try`

## 弱关键字

这类关键词只有在特定的上下文中才有特殊的意义。例如，可以声明名为 `union` 的变量或方法。

* `union` 用于声明[联合体]，它只有在联合体声明中使用时才是关键字。
* `'static` 用于静态生命周期，不能用作通用生命周期参数

  ```compile_fail
  // error[E0262]: invalid lifetime parameter name: `'static`
  fn invalid_lifetime_parameter<'static>(s: &'static str) -> &'static str { s }
  ```
* 在 2015 版本中，[`dyn`] 是在类型位置后面不是以::开头的路径中使用的关键字。从 2018 版开始，`dyn` 被提升为一个严格关键词。

> **<sup>词法分析器</sup>**\
> KW_UNION          : `union`\
> KW_STATICLIFETIME : `'static`
>
> **<sup>词法分析器 2015</sup>**\
> KW_DYN            : `dyn`

[项目]: items.md
[变量]: variables.md
[类型参数]: types/parameters.md
[循环标签]: expressions/loop-expr.md#loop-labels
[宏]: macros.md
[属性]: attributes.md
[宏占位符]: macros-by-example.md
[crate]: crates-and-source-files.md
[联合体]: items/unions.md
[枚举的变体]: items/enumerations.md
[`dyn`]: types/trait-object.md