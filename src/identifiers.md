# 标识符

>[notation.md](https://github.com/rust-lang/reference/blob/master/src/identifiers.md)\
>commit 34d27fe8bc8b89b55da690484d1e17fbd0f25055

> **<sup>词法分析器:<sup>**\
> IDENTIFIER_OR_KEYWORD :\
> &nbsp;&nbsp; &nbsp;&nbsp; [`a`-`z` `A`-`Z`]&nbsp;[`a`-`z` `A`-`Z` `0`-`9` `_`]<sup>\*</sup>\
> &nbsp;&nbsp; | `_` [`a`-`z` `A`-`Z` `0`-`9` `_`]<sup>+</sup>
>
> RAW_IDENTIFIER : `r#` IDENTIFIER_OR_KEYWORD <sub>*排除 `crate`, `self`, `super`, `Self`*</sub>
>
> NON_KEYWORD_IDENTIFIER : IDENTIFIER_OR_KEYWORD <sub>*排除一个[严格]或[保留]关键字 *</sub>
>
> IDENTIFIER :\
> NON_KEYWORD_IDENTIFIER | RAW_IDENTIFIER

标识符是如下形式的任何非空 ASCII 字符串：

要么是:

* 首字符是字母。
* 其余字符是字母、数字，或 `_`。

要么是：

* 首字符是 `_`。
* 标识符由多个字符组成，单个 `_` 不是有效标识符。
* 其余字符是字母、数字，或 `_`。

原生标识符与普通标识符类似，但前缀为 `r#`。（请注意 `r#` 前缀不包括在实际标识符中。）与普通标识符不同，原生标识符可以是除上面列出的 `RAW_IDENTIFIER` 之外的任何严格关键字或保留关键字。

[严格]: keywords.md#严格关键字
[保留]: keywords.md#保留关键字
