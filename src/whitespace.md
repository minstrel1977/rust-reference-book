# 空白

>[whitespace.md](https://github.com/rust-lang/reference/blob/master/src/whitespace.md)\
>commit 716e04f203c10387bd66aa3bcf1663e75ce208cf

空白是非空字符串，它里面只包含具有 [`Pattern_White_Space`] 属性的 Unicode 字符，即:

- `U+0009` (水平制表符, `'\t'`)
- `U+000A` (换行符, `'\n'`)
- `U+000B` (垂直制表符)
- `U+000C` (分页符)
- `U+000D` (回车符, `'\r'`)
- `U+0020` (空格符, `' '`)
- `U+0085` (下一行标记符)
- `U+200E` (从左到右标记符)
- `U+200F` (从右到标左记符)
- `U+2028` (行分隔符)
- `U+2029` (段分隔符)

Rust是一种“自由形式”的语言，这意味着所有形式的空白在语法中仅用于分隔*标记码*的作用，没有语义意义。

Rust 程序中，如果将一个空白元素替换为任何其他合法的空白元素(例如单个空格字符)，它们仍有相同的意义。

[`Pattern_White_Space`]: https://www.unicode.org/reports/tr31/
