# 标记符号

>[notation.md](https://github.com/rust-lang/reference/blob/master/src/notation.md)\
>commit b0e0ad6490d6517c19546b1023948986578fc378

## 语法

下表中的各种符号被用于 *词法* 和 *句法* 的语法片段：

| 符号           | 示例                      | 释义                                 
|-------------------|-------------------------------|--------------------------------|
| CAPITAL           | KW_IF, INTEGER_LITERAL        | 由词法分析生成的[标记码](token)|
| _ItalicCamelCase_ | _LetStatement_, _Item_        | 句法分析产生的内部语义                        |
| `string`          | `x`, `while`, `*`             | 确切的字符(串)                   |
| \\x               | \\n, \\r, \\t, \\0            | 转义字符                        |
| x<sup>?</sup>     | `pub`<sup>?</sup>             | 可选项                          |
| x<sup>\*</sup>    | _OuterAttribute_<sup>\*</sup> | x 重复零次或多次                  |
| x<sup>+</sup>     |  _MacroMatch_<sup>+</sup>     | x 重复一次或多次                  |
| x<sup>a..b</sup>  | HEX_DIGIT<sup>1..6</sup>      | x 重复 a 到 b 次                 |
| \|                | `u8` \| `u16`, Block \| Item  | 或                              |
| [ ]               | [`b` `B`]                     | 列举的任意字符                    |
| [ - ]             | [`a`-`z`]                     | a 到 z 范围内的任意字符(包括 a 和 z)|
| ~[ ]              | ~[`b` `B`]                    | 列举范围外的任意字符(序列)          |
| ~`string`         | ~`\n`, ~`*/`                  | 此字符序列外的任意字符(序列)        |
| ( )               | (`,` _Parameter_)<sup>?</sup> | 数据项分组(Groups items)       |

## 字符串表

语法中的一些规则-特别是[单目运算符]，[双目运算符]和[关键字]—会以简化形式给出：作为可打印字符串的列表。这些规则构成了关于[标记码]规则的一个子集，并被假定为提供解析器的词法分析阶段的结果。词法分析阶段由一个<abbr title="确定性有限自动机(Deterministic Finite Automaton)">DFA</abbr>驱动，对所有这些字符串表实体进行析取操作。

当语法中出现如 `等宽(monospace)` 这样的字符串时，它代表对这种字符串表中的单个成员的隐式引用。查阅[标记码]以获取更多信息。

[双目运算符]: expressions/operator-expr.md#arithmetic-and-logical-binary-operators
[关键字]: keywords.md
[标记码]: tokens.md
[单目运算符]: expressions/operator-expr.md#borrow-operators
