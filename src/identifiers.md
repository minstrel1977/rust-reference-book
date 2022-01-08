# Identifiers
# 标识符

>[identifiers.md](https://github.com/rust-lang/reference/blob/master/src/identifiers.md)\
>commit: 85031eb68b7418e49575efbf6e6c7fdad7f9f532 \
>本章译文最后维护日期：2022-01-08

> **<sup>词法分析:<sup>**\
> IDENTIFIER_OR_KEYWORD :\
> &nbsp;&nbsp; &nbsp;&nbsp; XID_Start XID_Continue<sup>\*</sup>\
> &nbsp;&nbsp; | `_` XID_Continue<sup>+</sup>
>
> RAW_IDENTIFIER : `r#` IDENTIFIER_OR_KEYWORD <sub>*排除 `crate`, `self`, `super`, `Self`*</sub>
>
> NON_KEYWORD_IDENTIFIER : IDENTIFIER_OR_KEYWORD <sub>*排除[严格关键字][strict]和[保留关键字][reserved] *</sub>
>
> IDENTIFIER :\
> NON_KEYWORD_IDENTIFIER | RAW_IDENTIFIER

<!-- When updating the version, update the UAX links, too. -->
标识符遵循 [Unicode标准附录31][UAX31] 中针对 Unicode 13.0版的规范，后面所述内容在此版本中均有备述。
这里举一些标识符的示例：

* `foo`
* `_identifier`
* `r#true`
* `Москва`
* `東京`

其中 UAX #31 中要求标识符使用的（产生式）参数如下：

* 起始字符 := [`XID_Start`]，外加一个下划线 (U+005F)
* 后续字符 := [`XID_Continue`]
* 中间字符 := 空

还有一个附加约束，即单个下划线字符不是标识符。

> **注意**: 以下划线开头的标识符通常用于表示有意不会被实际使用的标识符，且会使 `rustc` 对未被使用的警告静音。

如果标识符没有下面[原生标识符](#raw-identifiers)章节中描述的 `r#`前缀，那它不能是[严格关键字][strict]或[保留关键字][reserved]。

标识符中不允许使用零宽度非连接符（ZWNJ U+200C）和零宽度连接符（ZWJ U+200D）。

在下列情况下，标识符仅限于 [`XID_Start`] 和 [`XID_Continue`] 的ASCII子集：

* [`extern crate`]声明
* [路径][path]中引用的外部 crate名
* 从文件系统中载入的未被[`path`属性][`path` attribute]限定的[模块][Module]名
* 被 [`no_mangle`]属性限定的程序项
* [外部块][external blocks]中的程序项名称

## Normalization
## 标准化

标识符使用[Unicode标准附录15][UAX15]中定义的规范化形式C（NFC）进行规范化。如果两个标识符的 NFC形式相等，那么它们就是等价的。

[过程宏][proc macro]和[声明宏][mbe]在其输入中接受规范化的标识符。

## Raw identifiers
## 原生标识符

除了有形式前缀 `r#` 修饰外，原生标识符与普通标识符类似。（注意形式前缀 `r#` 不包括在实际标识符中。）与普通标识符不同，原生标识符可以是除上面列出的 `RAW_IDENTIFIER` 之外的任何严格关键字或保留关键字。

[`extern crate`]: items/extern-crates.md
[`no_mangle`]: abi.md#the-no_mangle-attribute
[`path` attribute]: items/modules.md#the-path-attribute
[`XID_Continue`]: http://unicode.org/cldr/utility/list-unicodeset.jsp?a=%5B%3AXID_Continue%3A%5D&abb=on&g=&i=
[`XID_Start`]:  http://unicode.org/cldr/utility/list-unicodeset.jsp?a=%5B%3AXID_Start%3A%5D&abb=on&g=&i=
[external blocks]: items/external-blocks.md
[mbe]: macros-by-example.md
[module]: items/modules.md
[path]: paths.md
[proc-macro]: procedural-macros.md
[reserved]: keywords.md#reserved-keywords
[strict]: keywords.md#strict-keywords
[UAX15]: https://www.unicode.org/reports/tr15/tr15-50.html
[UAX31]: https://www.unicode.org/reports/tr31/tr31-33.html