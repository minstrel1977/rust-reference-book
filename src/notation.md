# Notation
# 表义符/符号

>[notation.md](https://github.com/rust-lang/reference/blob/master/src/notation.md)\
>commit: dd1b9c331eb14ea7047ed6f2b12aaadab51b41d6 \
>本译文最后维护日期：2020-11-5

## Grammar
## 语法

本书中给出的 *词法* 和 *句法* 的语法片段会用到下表中的各种表义符：

| 表义符             | 示例                           | 释义                                 
|-------------------|-------------------------------|--------------------------------|
| CAPITAL           | KW_IF, INTEGER_LITERAL        | 由词法分析生成的标记码(token)      |
| _ItalicCamelCase_ | _LetStatement_, _Item_        | 句法生产式(syntactical production)|
| `string`          | `x`, `while`, `*`             | 确切的字面字符(串)                |
| \\x               | \\n, \\r, \\t, \\0            | 转义字符                         |
| x<sup>?</sup>     | `pub`<sup>?</sup>             | 可选项                           |
| x<sup>\*</sup>    | _OuterAttribute_<sup>\*</sup> | x 重复零次或多次                  |
| x<sup>+</sup>     | _MacroMatch_<sup>+</sup>      | x 重复一次或多次                  |
| x<sup>a..b</sup>  | HEX_DIGIT<sup>1..6</sup>      | x 重复 a 到 b 次                 |
| \|                | `u8` \| `u16`, Block \| Item  | 或                               |
| \[ ]              | \[`b` `B`]                    | 列举的任意字符                    |
| \[ - ]            | \[`a`-`z`]                    | a 到 z 范围内的任意字符(包括 a 和 z)|
| ~\[ ]             | ~\[`b` `B`]                   | 列举范围外的任意字符(序列)          |
| ~`string`         | ~`\n`, ~`*/`                  | 此字符序列外的任意字符(序列)        |
| ( )               | (`,` _Parameter_)<sup>?</sup> | 数据项分组                        |

## String table productions
## 字符串表示的句法生产式列表

语法中的一些规则 &mdash; 特别是[一元运算符][unary operators]，[二元运算符][binary operators]和[关键字][keywords] &mdash; 会以简化形式 - 作为可打印字符串的列表 - 给出。这些规则构成了[标记码][tokens]相关规则的规则子集，并且它们被假定为词法分析阶段的结果。词法分析阶段由<abbr title="确定性有限自动机(Deterministic Finite Automaton)">DFA</abbr>驱动，并应用这些规则来对源码进行析取操作。（译者注：原文在这里并没有给出句法生产式的列表，但本书会假设存在这么一个列表。）

本书还约定，当语法中出现如 `monospace` 这样的字符串时，它代表对这些生产式中的单个标记码成员的隐式引用。查阅[标记码][tokens]以获取更多信息。（译者注：如果译者觉得这种引用需要翻译时，会使用如：等宽(`monospace`) 这种形式来翻译，但读者需要意识到“monospace”是语言里的一个标记码，是以其字面形式出现在源码里的。）

[binary operators]: expressions/operator-expr.md#arithmetic-and-logical-binary-operators
[keywords]: keywords.md
[tokens]: tokens.md
[unary operators]: expressions/operator-expr.md#borrow-operators

<!-- 2020-11-3 -->
<!-- checked -->