# Identifiers
# 标识符

>[identifiers.md](https://github.com/rust-lang/reference/blob/master/src/identifiers.md)\
>commit: dd1b9c331eb14ea7047ed6f2b12aaadab51b41d6 \
>本译文最后维护日期：2020-11-5

> **<sup>词法分析:<sup>**\
> IDENTIFIER_OR_KEYWORD :\
> &nbsp;&nbsp; &nbsp;&nbsp; \[`a`-`z` `A`-`Z`]&nbsp;\[`a`-`z` `A`-`Z` `0`-`9` `_`]<sup>\*</sup>\
> &nbsp;&nbsp; | `_` \[`a`-`z` `A`-`Z` `0`-`9` `_`]<sup>+</sup>
>
> RAW_IDENTIFIER : `r#` IDENTIFIER_OR_KEYWORD <sub>*排除 `crate`, `self`, `super`, `Self`*</sub>
>
> NON_KEYWORD_IDENTIFIER : IDENTIFIER_OR_KEYWORD <sub>*排除[严格关键字][strict]和[保留关键字][reserved] *</sub>
>
> IDENTIFIER :\
> NON_KEYWORD_IDENTIFIER | RAW_IDENTIFIER

标识符是如下形式的任何非空 ASCII 字符串：

要么是:

* 首字符是字母。
* 其余字符是字母、数字，或 `_`。

要么是：

* 首字符是 `_`。
* 整个字符串由多个字符组成。单个 `_` 不是有效标识符。
* 其余字符是字母、数字，或 `_`。

除了有形式前缀 `r#` 修饰外，原生标识符(raw identifier)与普通标识符类似。（注意形式前缀 `r#` 不包括在实际标识符中。）与普通标识符不同，原生标识符可以是除上面列出的 `RAW_IDENTIFIER` 之外的任何严格关键字或保留关键字。

[strict]: keywords.md#strict-keywords
[reserved]: keywords.md#reserved-keywords

<!-- 2020-11-3 -->
<!-- checked -->
