# Tokens

>[tokens.md](https://github.com/rust-lang/reference/blob/master/src/tokens.md)\
>commit: 5afb503a4c1ea3c84370f8f4c08a1cddd1cdf6ad \
>本章译文最后维护日期：2024-03-09


token 是采用非递归方式的正则文法(regular languages)定义的基本语法产生式(primitive productions)。Rust 源码输入可以被分解成以下几类 token：

* [关键字][Keywords]
* [标识符][identifier]
* [字面量](#literals)
* [生存期](#lifetimes-and-loop-labels)
* [标点符号](#punctuation)
* [分隔符](#delimiters)

在本文档中，“简单”token 会直接在（相关章节头部的）[字符串表产生式(production)][string table production]表单中给出，并以 `monospace` 字体显示。（译者注：本译作的原文中，在文法表之外的行文中也会大量出现这种直接使用简单token 来替代相关名词的做法，一般此时如果译者觉得这种 token 需要翻译时，会使用诸如：结构体(`struct`) 这种形式来翻译。读者需要意识到“struct”是文法里的一个 token，能以其字面形式直接出现在源码里。）

## Literals
## 字面量

字面量是[字面量表达式][literal expressions]中使用的各种 token。

### Examples
### 示例

#### Characters and strings
#### 字符和字符串

|                                              | 举例         | `#` 号的数量[^nsets]  | 字符集  | 转义             |
|----------------------------------------------|-----------------|-------------|-------------|---------------------|
| [字符](#character-literals)| `'H'` | 0 | 全部 Unicode | [引号](#quote-escapes) & [ASCII](#ascii-escapes) & [Unicode](#unicode-escapes) |
| [字符串](#string-literals)| `"hello"`| 0 | 全部 Unicode | [引号](#quote-escapes) & [ASCII](#ascii-escapes) & [Unicode](#unicode-escapes)|
| [原生字符串](#raw-string-literals)| `r#"hello"#`    | <256 | 全部 Unicode | `N/A`                                                      |
| [字节](#byte-literals)| `b'H'`          | 0           | 全部 ASCII   | [引号](#quote-escapes) & [字节](#byte-escapes) |
| [字节串](#byte-string-literals)| `b"hello"`      | 0           | 全部 ASCII   | [引号](#quote-escapes) & [字节](#byte-escapes) |
| [原生字节串](#raw-byte-string-literals)| `br#"hello"#`   | <256 | 全部 ASCII   | `N/A`|
| [C语言风格的字符串](#c-string-literals)               | `c"hello"`      | 0          | 全部 Unicode | [Quote](#quote-escapes) & [Byte](#byte-escapes) & [Unicode](#unicode-escapes)   |
| [原生C语言风格的字符串](#raw-c-string-literals)       | `cr#"hello"#`   | <256       | 全部 Unicode | `N/A`                                                                           |

[^nsets]: 字面量两侧的 `#` 数量必须相同。

> **注意**: 字符和字符串字面量token 不会包括 `U+000D`(CR)后紧跟 `U+000A`(LF) 的字符序列：这对字符会在编译器读取源文件时被转换为单个 `U+000A`(LF)字符。

#### ASCII escapes
#### ASCII 转义

|   | 名称 |
|---|------|
| `\x41` | 7-bit 字符编码（2位数字，最大值为 `0x7F`）|
| `\n` | 换行符 |
| `\r` | 回车符 |
| `\t` | 制表符 |
| `\\` | 反斜线 |
| `\0` | Null |

#### Byte escapes
#### 字节转义

|   | 名称 |
|---|------|
| `\x7F` | 8-bit 字符编码（2位数字）|
| `\n` | 换行符 |
| `\r` | 回车符 |
| `\t` | 制表符 |
| `\\` | 反斜线 |
| `\0` | Null |

#### Unicode escapes
#### unicode 转义

|   | 名称 |
|---|------|
| `\u{7FFF}` | 24-bit Unicode 字符编码（最多6个数字）|

#### Quote escapes
#### 引号转义

|   | Name |
|---|------|
| `\'` | 单引号 |
| `\"` | 双引号 |

#### Numbers
#### 数字

| [数字字面量](#数字字面量)[^nl] | 示例 | 指数 |
|----------------------------------------|---------|----------------|
| 十进制整数 | `98_222` | `N/A` |
| 十六进制整数 | `0xff` | `N/A` |
| 八进制整数 | `0o77` | `N/A` |
| 二进制整数 | `0b1111_0000` | `N/A` |
| 浮点数 | `123.0E+77` | `Optional` |

[^nl]: 所有数字字面量允许使用 `_` 作为可视分隔符，比如：`1_234.0E+18f64`

#### Suffixes
#### 后缀

后缀是字面量主体部分后面的字符序列（它们之间不能含有空格），其形式与非原生标识符或关键字相同。

> **<sup>词法</sup>**\
> SUFFIX : IDENTIFIER_OR_KEYWORD\
> SUFFIX_NO_E : SUFFIX <sub>_not beginning with `e` or `E`_</sub>

任何带有后缀的字面量（如字符串、整型等）都可以作为有效的 token。

带有后缀的字面量token 可以传递给宏而不会产生错误。
宏自己决定如何解释这种 token，以及是否该报错。
特别是，声明宏的 `literal`段指示符匹配带有任意后缀的字面量token。

```rust
macro_rules! blackhole { ($tt:tt) => () }
macro_rules! blackhole_lit { ($l:literal) => () }

blackhole!("string"suffix); // OK
blackhole_lit!(1suffix); // OK
```

但是，那些被解析为字面量表达式或模式的字面量token 的后缀是受限的。
非数字文字标记上的任何后缀都将被拒绝，数字文字标记仅接受以下列表中的后缀。

| 整数 | 浮点数 |
|---------|----------------|
| `u8`, `i8`, `u16`, `i16`, `u32`, `i32`, `u64`, `i64`, `u128`, `i128`, `usize`, `isize` | `f32`, `f64` |
### Character and string literals
### 字符和字符串字面量

#### Character literals
#### 字符字面量

> **<sup>词法</sup>**\
> CHAR_LITERAL :\
> &nbsp;&nbsp; `'` ( ~\[`'` `\` \\n \\r \\t] | QUOTE_ESCAPE | ASCII_ESCAPE | UNICODE_ESCAPE ) `'` SUFFIX<sup>?</sup>
>
> QUOTE_ESCAPE :\
> &nbsp;&nbsp; `\'` | `\"`
>
> ASCII_ESCAPE :\
> &nbsp;&nbsp; &nbsp;&nbsp; `\x` OCT_DIGIT HEX_DIGIT\
> &nbsp;&nbsp; | `\n` | `\r` | `\t` | `\\` | `\0`
>
> UNICODE_ESCAPE :\
> &nbsp;&nbsp; `\u{` ( HEX_DIGIT `_`<sup>\*</sup> )<sup>1..6</sup> `}`

*字符字面量*是位于两个 `U+0027`（单引号 `'`）字符内的单个 Unicode 字符。当它是 `U+0027` 自身时，必须前置*转义*字符 `U+005C`（`\`）。

#### String literals
#### 字符串字面量

> **<sup>词法</sup>**\
> STRING_LITERAL :\
> &nbsp;&nbsp; `"` (\
> &nbsp;&nbsp; &nbsp;&nbsp; ~\[`"` `\` _IsolatedCR_]&nbsp;&nbsp;(译者注：IsolatedCR：后面没有跟 `\n` 的 `\r`，首次定义见[注释](comments.md))\
> &nbsp;&nbsp; &nbsp;&nbsp; | QUOTE_ESCAPE\
> &nbsp;&nbsp; &nbsp;&nbsp; | ASCII_ESCAPE\
> &nbsp;&nbsp; &nbsp;&nbsp; | UNICODE_ESCAPE\
> &nbsp;&nbsp; &nbsp;&nbsp; | STRING_CONTINUE\
> &nbsp;&nbsp; )<sup>\*</sup> `"` SUFFIX<sup>?</sup>
>
> STRING_CONTINUE :\
> &nbsp;&nbsp; `\` _后跟_ \\n

*字符串字面量*是位于两个 `U+0022` （双引号 `"`）字符内的任意 Unicode 字符序列。当它是 `U+0022` 自身时，必须前置*转义*字符 `U+005C`（`\`）。

字符串字面量允许使用字符`U+000A`(LF) 的形式来换行书写。
但当非转义的字符 `U+005C`（`\`）后面紧跟着一个换行符时，换行符并不会出现在字符串中。
详细信息请参见[字符串接续符转义][String continuation escapes]。
字符`U+000D`(CR) 除了作为这种字符串接续符转义的一部分之外，不能出现在字符串字面量中。

#### Character escapes
#### 字符转义

不管是字符字面量，还是非原生字符串字面量，Rust 都为其提供了额外的*转义*功能。转义以一个 `U+005C`（`\`）开始，并后跟如下形式之一：

* *7-bit 码点转义*以 `U+0078`（`x`）开头，后面紧跟两个*十六进制数字*，其最大值为 `0x7F`。它表示 ASCII 字符，其码值就等于字面提供的十六进制值。不允许使用更大的值，因为不能确定其是 Unicode 码点还是字节值(byte values)。
* *24-bit 码点转义*以 `U+0075`（`u`）开头，后跟多达六位*十六进制数字*，位于花括号 `U+007B`（`{`）和 `U+007D`（`}`）之间。这表示（需转义到的）Unicode 字符的码点等于花括号里的十六进制值。
* *空白符转义*是 `U+006E` (`n`)、`U+0072` (`r`) 或者 `U+0074` (`t`) 之一，依次分别表示 Unicode 码点 `U+000A`（LF），`U+000D`（CR），或者 `U+0009`（HT）。
* *null转义* 是字符 `U+0030`（`0`），表示 Unicode 码点 `U+0000`（NUL）。
* *反斜线转义* 是字符 `U+005C`（`\`），反斜线必须通过转义才能表示其自身。

#### Raw string literals
#### 原生字符串字面量

> **<sup>词法</sup>**\
> RAW_STRING_LITERAL :\
> &nbsp;&nbsp; `r` RAW_STRING_CONTENT SUFFIX<sup>?</sup>
>
> RAW_STRING_CONTENT :\
> &nbsp;&nbsp; &nbsp;&nbsp; `"` ( ~ _IsolatedCR_ )<sup>* (非贪婪模式)</sup> `"`\
> &nbsp;&nbsp; | `#` RAW_STRING_CONTENT `#`

原生字符串字面量不做任何转义。它以字符 `U+0072`（`r`）后跟小于256个字符 `U+0023`（`#`），以及一个字符 `U+0022`（双引号 `"`），这样的字符组合开始。
中间*原生字符串主体*部分可包含除了 `U+000D`(CR) 之外的任意 Unicode 字符序列。
最后再后跟另一个 `U+0022`（双引号 `"`）以及跟与字符串主体前的那段字符组合中的同等数量的 `U+0023`（`#`）的字符来表示字符串主体的结束。

所有包含在原生字符串文本主体中的 Unicode 字符都代表他们自身：字符 `U+0022`（双引号 `"`）（除非后跟的纯 `U+0023` (`#`)字符串与文本主体开始前的对称相等）或字符 `U+005C`（`\`）此时都没有特殊含义。

字符串字面量示例:

```rust
"foo"; r"foo";                     // foo
"\"foo\""; r#""foo""#;             // "foo"

"foo #\"# bar";
r##"foo #"# bar"##;                // foo #"# bar

"\x52"; "R"; r"R";                 // R
"\\x52"; r"\x52";                  // \x52
```

### Byte and byte string literals
### 字节和字节串字面量

#### Byte literals
#### 字节字面量

> **<sup>词法</sup>**\
> BYTE_LITERAL :\
> &nbsp;&nbsp; `b'` ( ASCII_FOR_CHAR | BYTE_ESCAPE )  `'` SUFFIX<sup>?</sup>
>
> ASCII_FOR_CHAR :\
> &nbsp;&nbsp; _任何 ASCII 字符 （0x00 到 0x7F）, 排除_ `'`, `\`, \\n, \\r 或者 \\t
>
> BYTE_ESCAPE :\
> &nbsp;&nbsp; &nbsp;&nbsp; `\x` HEX_DIGIT HEX_DIGIT\
> &nbsp;&nbsp; | `\n` | `\r` | `\t` | `\\` | `\0` | `\'` | `\"`

*字节字面量*是单个 ASCII 字符（码值在 `U+0000` 到 `U+007F` 区间内）或一个*转义字节*作为字节字面量的真实主体跟在表示形式意义的字符 `U+0062`（`b`）和字符 `U+0027`（单引号 `'`）组合之后，然后再后接字符 `U+0027`。如果字符 `U+0027` 本身要出现在字面量中，它必须经由前置字符 `U+005C`（`\`）*转义*。字节字面量等价于一个 `u8` 8-bit 无符号整型*数字字面量*。

#### Byte string literals
#### 字节串字面量

> **<sup>词法</sup>**\
> BYTE_STRING_LITERAL :\
> &nbsp;&nbsp; `b"` ( ASCII_FOR_STRING | BYTE_ESCAPE | STRING_CONTINUE )<sup>\*</sup> `"` SUFFIX<sup>?</sup>
>
> ASCII_FOR_STRING :\
> &nbsp;&nbsp; _任何 ASCII 字符(码值位于 0x00 到 0x7F 之间), 排除_ `"`, `\` _和 IsolatedCR_

非原生*字节串字面量*是 ASCII 字符和转义字符组成的字符序列，形式是以字符 `U+0062`（`b`）和字符 `U+0022`（双引号 `"`）组合开头，以字符 `U+0022` 结尾。如果字面量中包含字符 `U+0022`，则必须由前置的 `U+005C`（`\`）_转义_。
或者，字节串字面量也可以是*原生字节串字面量*（下面有其定义）。

字节串字面量允许使用字符`U+000A`(LF) 的形式来换行书写。
但当非转义的字符 `U+005C`（`\`）后面紧跟着一个换行符时，换行符并不会出现在字符串中。
详细信息请参见[字符串接续符转义][String continuation escapes]。
字符`U+000D`(CR) 除了作为这种字符串接续符转义的一部分之外，不能出现在字节串字面量中。

一些额外的*转义*可以在字节或非原生字节串字面量中使用，转义以 `U+005C`（`\`）开始，并后跟如下形式之一：

* *字节转义*以 `U+0078` (`x`)开始，后跟恰好两位*十六进制数字*来表示十六进制值代表的字节。
* *空白符转义*是字符 `U+006E`（`n`）、`U+0072`（`r`），或 `U+0074`（`t`）之一，分别表示字节值 `0x0A`（ASCII LF）、`0x0D`（ASCII CR），或 `0x09`（ASCII HT）。
* *null转义*是字符 `U+0030`（`0`），表示字节值 `0x00` （ASCII NUL）。
* *反斜线转义*是字符 `U+005C`（`\`），必须被转义以表示其 ASCII 编码 `0x5C`。

#### Raw byte string literals
#### 原生字节串字面量

> **<sup>词法</sup>**\
> RAW_BYTE_STRING_LITERAL :\
> &nbsp;&nbsp; `br` RAW_BYTE_STRING_CONTENT SUFFIX<sup>?</sup>
>
> RAW_BYTE_STRING_CONTENT :\
> &nbsp;&nbsp; &nbsp;&nbsp; `"` ASCII_FOR_RAW<sup>* (非贪婪模式)</sup> `"`\
> &nbsp;&nbsp; | `#` RAW_BYTE_STRING_CONTENT `#`
>
> ASCII_FOR_RAW :\
> &nbsp;&nbsp; _除了 IsolatedCR 外的任何 ASCII 字符（0x00 到 0x7F）_

原生字节串字面量不做任何转义。它们以字符 `U+0062`（`b`）后跟 `U+0072`（`r`），再后跟小于256个字符 `U+0023`（`#`）及字符 `U+0022`（双引号 `"`），这样的字符组合开始。
中间是*原生字节串主体*，这部分可包含除了 `U+000D`(CR) 外的任意的 ASCII 字符序列。
最后再后跟另一个 `U+0022`（双引号 `"`）以及跟与字符串主体前的那段字符组合中的同等数量的 `U+0023`（`#`）的字符来表示字符串主体的结束。
原生字节串字面量不能包含任何非 ASCII 字节。

原生字节串文本主体中的所有字符都代表它们自身的 ASCII 编码，字符 `U+0022`（双引号 `"`）（除非后跟的纯 `U+0023`（`#`）字符串与文本主体开始前的对称相等）或字符 `U+005C`（`\`）此时都没有特殊含义。

字节串字面量示例：

```rust
b"foo"; br"foo";                     // foo
b"\"foo\""; br#""foo""#;             // "foo"

b"foo #\"# bar";
br##"foo #"# bar"##;                 // foo #"# bar

b"\x52"; b"R"; br"R";                // R
b"\\x52"; br"\x52";                  // \x52
```

### C string and raw C string literals
### C语言风格的字符串字面量和原生C语言风格的字符串字面量

#### C string literals
#### C语言风格的字符串字面量

> **<sup>词法</sup>**\
> C_STRING_LITERAL :\
> &nbsp;&nbsp; `c"` (\
> &nbsp;&nbsp; &nbsp;&nbsp; ~\[`"` `\` _IsolatedCR_ _NUL_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | BYTE_ESCAPE _except `\0` or `\x00`_\
> &nbsp;&nbsp; &nbsp;&nbsp; | UNICODE_ESCAPE _except `\u{0}`, `\u{00}`, …, `\u{000000}`_\
> &nbsp;&nbsp; &nbsp;&nbsp; | STRING_CONTINUE\
> &nbsp;&nbsp; )<sup>\*</sup> `"` SUFFIX<sup>?</sup>

_C语言风格的字符串字面量_是通过前面是字符`U+0063` (`c`) 和 `U+0022`（双引号），后面是字符`U+0022` 转义过的 Unicode字符序列。如果字面量中存在字符`U+0022`，则其前面必须用`U+005C` (`\`)字符进行转义。
或者，C语言风格的字符串字面量可以是下面定义的_原生C语言风格的字符串字面量_。

[CStr]: ../core/ffi/struct.CStr.html

C语言风格的字符串由字节`0x00`隐式终止，因此C语言风格的字符串字面量`c""`等同于从字节字符串文字`b"\x00"`来手动构造一个`&CStr`。除了隐式终止符之外，C语言风格的字符串中不允许再使用字节`0x00`。

C语言风格的字符串字面量允许使用字符`U+000A`(LF) 的形式来换行书写。
但当非转义的字符 `U+005C`（`\`）后面紧跟着一个换行符时，换行符并不会出现在字符串中。
详细信息请参见[字符串接续符转义][String continuation escapes]。
字符`U+000D`(CR) 除了作为这种字符串接续符转义的一部分之外，不能出现在C语言风格的字符串字面量中。

一些额外的转义符在非原生C语言风格的字符串字面量中可用。转义以`U+005C` (`\`)开头，并后继以下形式之一：

* 一个_字节转义符_以`U+0078` (`x`)开头，后面正好跟两个_十六进制数字_。它表示等于所提供的十六进制值的字节序。
* 一个_24位码点转义符_以 `U+0075` (`u`) 开头，后面最多有六个由大括号`U+007B` (`{`)和`U+007D` (`}`)包围的_十六进制数字_。它表示由这些十六进制数值通过的UTF-8编码的Unicode码点值。
* _空白转义符_是字符`U+006E` (`n`)、`U+0072` (`r`) 或 `U+0074` (`t`) 之一，分别表示字节`0x0A` (ASCII LF)、`0x0D` (ASCII CR) 或 `0x09` (ASCII HT)。
* _反斜杠转义符_就是字符`U+005C` (`\`)，必须对其进行转义才能表示其ASCII编码的`0x5C`。

C语言风格的字符串本身表示没有定义编码类型的字节序，但 C语言风格的字符串字面量又可能包含`U+007F`以上的 Unicode字符。这种情况下这些字符将被替换为该字符的UTF-8表示形式的字节。

以下 C语言风格的字符串字面量表达形式等效：

```rust
c"æ";        // 小写的拉丁字符 AE (U+00E6)
c"\u{00E6}";
c"\xC3\xA6";
```

> **版次差异**: 2021版或更高版本中可以使用 C语言风格的字符串字面量。在更低的版次中，`c""`token 会被词法解析器解析为 `c ""`。

#### Raw C string literals
#### 原生C语言风格的字符串字面量

> **<sup>Lexer</sup>**\
> RAW_C_STRING_LITERAL :\
> &nbsp;&nbsp; `cr` RAW_C_STRING_CONTENT SUFFIX<sup>?</sup>
>
> RAW_C_STRING_CONTENT :\
> &nbsp;&nbsp; &nbsp;&nbsp; `"` ( ~ _IsolatedCR_ _NUL_ )<sup>* (non-greedy)</sup> `"`\
> &nbsp;&nbsp; | `#` RAW_C_STRING_CONTENT `#`

原生C语言风格的字符串字面量不处理任何转义。它们以字符`U+0063` (`c`)开头，然后跟`U+0072` (`r`)，然后再跟少于256个的字符`U+0023` (`#`)和一个 `U+0022`（双引号）字符（记作开头引号）。
中间_原生C语言风格的字符串本体_可以包含除 `U+0000`(NUL) 和 `U+000D`(CR) 之外的任何 Unicode字符序列。
最后再后跟另一个 `U+0022`（双引号 `"`）以及跟与字符串主体前的那段字符组合中的同等数量的 `U+0023`（`#`）的字符来表示字符串主体的结束。

原生C语言风格的字符串本体中包含的所有字符都以 UTF-8编码形式表示。字符`U+0022`（双引号）（后面跟有至少与用于开始原生C语言风格的字符串字面量的数量相同的 `U+0023` (`#`)字符时除外）或 `U+005C` (`\`) 没有任何特殊含义。

> **版次差异**: 2021版或更高版次可以使用原生C语言风格的字符串字面量。在更低的版次中，`cr""`token 会被词法解析器解析为`cr ""` ，`cr#""#` 被解析为 `cr #""#`（这是不符合语法的）。

#### Examples for C string and raw C string literals
#### C语言风格的字符串字面量和原生C语言风格的字符串字面量的示例

```rust
c"foo"; cr"foo";                     // foo
c"\"foo\""; cr#""foo""#;             // "foo"

c"foo #\"# bar";
cr##"foo #"# bar"##;                 // foo #"# bar

c"\x52"; c"R"; cr"R";                // R
c"\\x52"; cr"\x52";                  // \x52
```

### Number literals
### 数字字面量

*数字字面量*可以是*整型字面量*，也可以是*浮点型字面量*，识别这两种字面量的文法是混合在一起的。

#### Integer literals
#### 整型字面量

> **<sup>词法</sup>**\
> INTEGER_LITERAL :\
> &nbsp;&nbsp; ( DEC_LITERAL | BIN_LITERAL | OCT_LITERAL | HEX_LITERAL )
>              SUFFIX_NO_E<sup>?</sup>
>
> DEC_LITERAL :\
> &nbsp;&nbsp; DEC_DIGIT (DEC_DIGIT|`_`)<sup>\*</sup>
>
> BIN_LITERAL :\
> &nbsp;&nbsp; `0b` (BIN_DIGIT|`_`)<sup>\*</sup> BIN_DIGIT (BIN_DIGIT|`_`)<sup>\*</sup>
>
> OCT_LITERAL :\
> &nbsp;&nbsp; `0o` (OCT_DIGIT|`_`)<sup>\*</sup> OCT_DIGIT (OCT_DIGIT|`_`)<sup>\*</sup>
>
> HEX_LITERAL :\
> &nbsp;&nbsp; `0x` (HEX_DIGIT|`_`)<sup>\*</sup> HEX_DIGIT (HEX_DIGIT|`_`)<sup>\*</sup>
>
> BIN_DIGIT : \[`0`-`1`]
>
> OCT_DIGIT : \[`0`-`7`]
>
> DEC_DIGIT : \[`0`-`9`]
>
> HEX_DIGIT : \[`0`-`9` `a`-`f` `A`-`F`]

*整型字面量*具备下述 4 种形式之一：

* *十进制字面量*以*十进制数字*开头，后跟*十进制数字*和*下划线(`_`)*的任意组合。
* *十六进制字面量*以字符序列 `U+0030` `U+0078`（`0x`）开头，后跟十六进制数字和下划线的任意组合（至少一个数字）。
* *八进制字面量*以字符序列 `U+0030` `U+006F`（`0o`）开头，后跟八进制数字和下划线的任意组合（至少一个数字）。
* *二进制字面量*以字符序列 `U+0030` `U+0062`（`0b`）开头，后跟二进制数字和下划线的任意组合（至少一个数字）。

与其它字面量一样，整型字面量后面可紧跟（没有空格）一个上述的后缀。
后缀不能以 `e` 或 `E` 开头，因为这将被解析为浮点字面量的指数。
参见[整型字面量表达式][Integer literal expressions]以了解这些后缀的功能效果。

被正确解析为整型字面量的示例：

```rust
# #![allow(overflowing_literals)]
123;
123i32;
123u32;
123_u32;

0xff;
0xff_u8;
0x01_f32; // 注意这是整数 7986, 不是浮点数 1.0
0x01_e3;  // 注意这是整数 483, 不是浮点数 1000.0

0o70;
0o70_i16;

0b1111_1111_1001_0000;
0b1111_1111_1001_0000i64;
0b________1;

0usize;

// 下面这些对它们的类型来说太大了，但仍被认为是字面量表达式
128_i8;
256_u8;

// 这是一个整型字面量，但被解析器接受为浮点型字面量表达式
5f32;

```

注意对于 `-1i8` 这样的，其实它被分析为两个 token: `-` 后跟 `1i8`。

不被承认为合法字面量表达式的整型字面量：

```rust
# #[cfg(FALSE)] {
0invalidSuffix;
123AFB43;
0b010a;
0xAB_CD_EF_GH;
0b1111_f32;
# }
```

#### Tuple index
#### 元组索引

> **<sup>词法</sup>**\
> TUPLE_INDEX: \
> &nbsp;&nbsp; INTEGER_LITERAL

元组索引用于引用[元组][tuples]、[元组结构体][tuple structs]和[元组变体][tuple variants]的字段。

元组索引直接与字面量token 进行比较。元组索引以 `0` 开始，每个后续索引的值以十进制的 `1` 递增。因此，元组索引只能匹配十进制值，并且该值不能用 `0` 做前缀字符。

```rust,compile_fail
let example = ("dog", "cat", "horse");
let dog = example.0;
let cat = example.1;
// 下面的示例非法.
let cat = example.01;  // 错误：没有 `01` 字段
let horse = example.0b10;  // 错误：没有 `0b10` 字段
```

> **注意**: 元组的索引可能包含一些特定的后缀，但是这不是被故意设计为有效的，可能会在将来的版本中被移除。
> 更多信息请参见<https://github.com/rust-lang/rust/issues/60210>。

#### Floating-point literals
#### 浮点型字面量

> **<sup>词法</sup>**\
> FLOAT_LITERAL :\
> &nbsp;&nbsp; &nbsp;&nbsp; DEC_LITERAL `.`
>   _（紧跟着的不能是 `.`, `_` 或者 XID_Start类型的字符)__\
> &nbsp;&nbsp; | DEC_LITERAL `.` DEC_LITERAL SUFFIX_NO_E<sup>?</sup>\
> &nbsp;&nbsp; | DEC_LITERAL (`.` DEC_LITERAL)<sup>?</sup> FLOAT_EXPONENT SUFFIX<sup>?</sup>
>
> FLOAT_EXPONENT :\
> &nbsp;&nbsp; (`e`|`E`) (`+`|`-`)<sup>?</sup>
>               (DEC_DIGIT|`_`)<sup>\*</sup> DEC_DIGIT (DEC_DIGIT|`_`)<sup>\*</sup>
>

*浮点型字面量*有如下两种形式：

* *十进制字面量*后跟句点字符 `U+002E` (`.`)。后面可选地跟着另一个十进制数字，还可以再接一个可选的*指数*。
* *十进制字面量*后跟一个*指数*。

如同整型字面量，浮点型字面量也可后跟一个后缀，但在后缀之前，浮点型字面量部分不以 `U+002E`（`.`）结尾。
如果字面量不包含指数，则后缀不能以 `e`或 `E` 开头。
参见[浮点型字面量表达式][Floating-point literal expressions]以了解这类后缀的功能效果。

各种形式的浮点型字面量示例：

```rust
123.0f64;
0.1f64;
0.1f32;
12E+99_f64;
let x: f64 = 2.;
```

最后一个例子稍显不同，因为不能对一个以句点结尾的浮点型字面量使用后缀句法，`2.f64` 会尝试在 `2` 上调用名为 `f64` 的方法。

请注意，像 `-1.0` 这样的会被分析为两个 token： `-` 后跟 `1.0`。

不被认为是合法的字面量表达式的浮点型字面量的示例：

```rust
# #[cfg(FALSE)] {
2.0f80;
2e5f80;
2e5e6;
2.0e5e6;
1.3e10u64;
# }
```

#### Reserved forms similar to number literals
#### 类似于数字字面量的保留形式

> **<sup>Lexer</sup>**\
> RESERVED_NUMBER :\
> &nbsp;&nbsp; &nbsp;&nbsp; BIN_LITERAL \[`2`-`9`&ZeroWidthSpace;]\
> &nbsp;&nbsp; | OCT_LITERAL \[`8`-`9`&ZeroWidthSpace;]\
> &nbsp;&nbsp; | ( BIN_LITERAL | OCT_LITERAL | HEX_LITERAL ) `.` \
> &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; _(不能直接后跟 `.`, `_` 或一个 XID_Start类型的字符)_\
> &nbsp;&nbsp; | ( BIN_LITERAL | OCT_LITERAL ) (`e`|`E`)\
> &nbsp;&nbsp; | `0b` `_`<sup>\*</sup> _end of input or not BIN_DIGIT_\
> &nbsp;&nbsp; | `0o` `_`<sup>\*</sup> _end of input or not OCT_DIGIT_\
> &nbsp;&nbsp; | `0x` `_`<sup>\*</sup> _end of input or not HEX_DIGIT_\
> &nbsp;&nbsp; | DEC_LITERAL ( . DEC_LITERAL)<sup>?</sup> (`e`|`E`) (`+`|`-`)<sup>?</sup> _end of input or not DEC_DIGIT_

后面词法形式和数字字面量差不多的*保留形式*。
由于这些可能会引起歧义，它们会被 token转化器(tokenizer)拒绝，而不是被解释为单独的 token。

* 不带后缀的二进制或八进制字面量，不插入空格的后跟一个超出其进制数字字符范围的十进制数字。

* 不带后缀的二进制、八进制或十六进制字面量，不插入空格的后跟一个句点字符（句点后面的内容与浮点数字面量相同）。

* 不带前缀的二进制或八进制字面量，不加空格的后跟字符`e`或`E`。

* 以一个进制数前缀开始的输入，但又不是有效的二进制、八进制或十六进制字面量（因为它没包含数字）。

* 具有浮点型字面量形式且指数中没有数字的输入。

这些保留形式的示例：

```rust,compile_fail
0b0102;  // 这可不是 `0b010` 后跟 `2`
0o1279;  // 这可不是 `0o127` 后跟 `9`
0x80.0;  // 这可不是 `0x80` 后跟 `.` and `0`
0b101e;  // 这不是一个带后缀的字面量，也不是 `0b101` 后跟 `e`
0b;      // 这不是一个整型字面量，也不是 `0` 后跟  `b`
0b_;     // 这不是一个整型字面量，也不是 `0` 后跟  `b_`
2e;      // 这不是一个浮点型字面量，也不是 `2` 后跟 `e`
2.0e;    // 这不是一个浮点型字面量，也不是 `2.0` 后跟 `e`
2em;     // 这不是一个带后缀的字面量，也不是 `2` 后跟 `em`
2.0em;   // 这不是一个带后缀的字面量，也不是 `2.0` 后跟 `em`
```

## Lifetimes and loop labels
## 生存期和循环标签

> **<sup>词法</sup>**\
> LIFETIME_TOKEN :\
> &nbsp;&nbsp; &nbsp;&nbsp; `'` [IDENTIFIER_OR_KEYWORD][identifier]
>   _(后面没有紧跟 `'`)_\
> &nbsp;&nbsp; | `'_`
>   _(后面没有紧跟 `'`)_
>
> LIFETIME_OR_LABEL :\
> &nbsp;&nbsp; &nbsp;&nbsp; `'` [NON_KEYWORD_IDENTIFIER][identifier]
>   _(后面没有紧跟 `'`)_

生存期参数和[循环标签][loop labels]使用 LIFETIME_OR_LABEL 类型的 token。（尽管 LIFETIME_OR_LABEL 是 LIFETIME_TOKEN 的子集，但）任何符合 LIFETIME_TOKEN 约定的 token 也都能被上述词法分析规则所接受，比如 LIFETIME_TOKEN 类型的 token 在宏中就可以畅通无阻的使用。

## Punctuation
## 标点符号

为了完整起见，这里列出了（Rust 里）所有的标点符号的 symbol token。它们各自的用法和含义在链接页面中都有定义。

| 符号 | 名称        | 使用方法 |
|--------|-------------|-------|
| `+`    | Plus        | [算术加法][arith], [trait约束][Trait Bounds], [可匹配空的宏匹配器][macros](Macro Kleene Matcher)
| `-`    | Minus       | [算术减法][arith], [取反][Negation]
| `*`    | Star        | [算术乘法][arith], [解引用][Dereference], [裸指针][Raw Pointers], [可匹配空的宏匹配器][macros], [use 通配符][Use wildcards]
| `/`    | Slash       | [算术除法][arith]
| `%`    | Percent     | [算术取模][arith]
| `^`    | Caret       | [位和逻辑异或][arith]
| `!`    | Not         | [位和逻辑非][Negation], [宏调用][macros], [内部属性][attributes], [never型][Never Type], [否定实现][negative impls]
| `&`    | And         | [位和逻辑与][arith], [借用][Borrow], [引用][References], [引用模式][Reference patterns]
| <code>\|</code> | Or | [位和逻辑或][arith], [闭包][Closures], [match] 中的模式, [if let], 和 [while let]
| `&&`   | AndAnd      | [短路与][lazy-bool], [借用][Borrow], [引用][References], [引用模式][Reference patterns]
| <code>\|\|</code> | OrOr | [短路或][lazy-bool], [闭包][Closures]
| `<<`   | Shl         | [左移位][arith], [嵌套泛型][generics]
| `>>`   | Shr         | [右移位][arith], [嵌套泛型][generics]
| `+=`   | PlusEq      | [加法及赋值][compound]
| `-=`   | MinusEq     | [减法及赋值][compound]
| `*=`   | StarEq      | [乘法及赋值][compound]
| `/=`   | SlashEq     | [除法及赋值][compound]
| `%=`   | PercentEq   | [取模及赋值][compound]
| `^=`   | CaretEq     | [按位异或及赋值][compound]
| `&=`   | AndEq       | [按位与及赋值][compound]
| <code>\|=</code> | OrEq | [按位或及赋值][compound]
| `<<=`  | ShlEq       | [左移位及赋值][compound]
| `>>=`  | ShrEq       | [右移位及赋值][compound], [嵌套泛型][generics]
| `=`    | Eq          | [赋值][Assignment], [属性][attributes], 各种类型定义
| `==`   | EqEq        | [等于][comparison]
| `!=`   | Ne          | [不等于][comparison]
| `>`    | Gt          | [大于][comparison], [泛型][generics], [路径][Paths]
| `<`    | Lt          | [小于][comparison], [泛型][generics], [路径][Paths]
| `>=`   | Ge          | [大于或等于][comparison], [泛型][generics]
| `<=`   | Le          | [小于或等于][comparison]
| `@`    | At          | [子模式绑定][Subpattern binding]
| `_`    | Underscore  | [通配符模式][Wildcard patterns], [自动推断型类型][Inferred types], [常量项][constants]中的非命名程序项, [外部 crate][extern crates], 和 [use声明][use declarations]，和[解构赋值][destructuring assignment]
| `.`    | Dot         | [字段访问][field], [元组索引][Tuple index]
| `..`   | DotDot      | [区间][range], [结构体表达式][Struct expressions], [模式][Patterns],[区间模式][rangepat]
| `...`  | DotDotDot   | [可变参数函数][extern], [区间模式][Range patterns]
| `..=`  | DotDotEq    | [闭区间][range], [区间模式][Range patterns]
| `,`    | Comma       | 各种分隔符
| `;`    | Semi        | 各种程序项和语句的结束符, [数组类型][Array types]
| `:`    | Colon       | 各种分隔符
| `::`   | PathSep     | [路径分隔符][路径][Paths]
| `->`   | RArrow      | [函数返回类型][functions], [闭包返回类型][Closures], [数组指针类型][Function pointer type]
| `=>`   | FatArrow    | [匹配臂][match], [宏][macros]
| `<-`   | LArrow      | 左箭头符号在Rust 1.0之后就没有再使用过，但它仍然被视为一个单独的 token
| `#`    | Pound       | [属性][attributes]
| `$`    | Dollar      | [宏][macros]
| `?`    | Question    | [问号运算符][question], [非确定性内存宽度][sized], [可匹配空的宏匹配器][macros]
| `~`    | Tilde       | 从 Rust 1.0 开始，波浪号操作符就弃用了，但其 token 可能仍在使用

## Delimiters
## 定界符

括号用于文法的各个部分，左括号必须始终与右括号配对。括号以及其内的 token 在[宏][macros]中被称作“token树(token trees)”。括号有三种类型：

| 括号 | 类型            |
|---------|-------------|
| `{` `}` | 花/大括号    |
| `[` `]` | 方/中括号    |
| `(` `)` | 圆/小括号    |

## Reserved prefixes
## 保留前缀

> **<sup>词法 2021+</sup>**\
> RESERVED_TOKEN_DOUBLE_QUOTE : ( IDENTIFIER_OR_KEYWORD <sub>_排除 `b` 或 `c` 或 `r` 或 `br` 或 `cr`_</sub> | `_` ) `"`\
> RESERVED_TOKEN_SINGLE_QUOTE : ( IDENTIFIER_OR_KEYWORD <sub>_排除 `b`_</sub> | `_` ) `'`\
> RESERVED_TOKEN_POUND : ( IDENTIFIER_OR_KEYWORD <sub>_排除 `r` 或 `br` 或 `cr`_</sub> | `_` ) `#`

这种被称为 _保留前缀_ 的词法形式是保留供将来使用的。

如果输入的源代码在词法上被解释为非原生标识符（或关键字或 `_`），且紧接着的字符是 `#`、`'` 或 `"` （没有空格），则将其标识为保留前缀。（译者注：例如 prefix#identifier, prefix"string", prefix'c', 和 prefix#123（其中 prefix可以是任何标识符（但不能是原生标识符））这样词法形式被标识为保留词法，以供将来拓展语法使用）

请注意，原生标识符、原生字符串字面量和原生字节字符串字面量可能包含 `#`字符，但不会被解释为包含保留前缀。

类似地，原生字符串字面量、字节字面量、字节字符串字面量、原生字节字符串字面量、C语言风格的字符串字面量和原生C语言风格的字符串字面量中使用的`r`、`b`、 `br`、 `c` 和 `cr` 前缀也不会解释为保留前缀。

> **版次差异**：从2021版开始，保留前缀被词法解释器报告为错误（特别是，它们不能再传递给宏了）。
>
> 在2021版之前，保留前缀可以被词法解释器接受并被解释为多个 token（例如，后接 `#` 的标识符或关键字）。
>
> 在所有的版次中都可以被编译器接受的示例：
> ```rust
> macro_rules! lexes {($($_:tt)*) => {}}
> lexes!{a #foo}
> lexes!{continue 'foo}
> lexes!{match "..." {}}
> lexes!{r#let#foo}         // 3个 tokens: r#let # foo
> ```
>
> 在 2021之前的版次中可以被编译器接受，但之后会被拒绝的示例：
> ```rust,edition2018
> macro_rules! lexes {($($_:tt)*) => {}}
> lexes!{a#foo}
> lexes!{continue'foo}
> lexes!{match"..." {}}
> ```

[Inferred types]: types/inferred.md
[Range patterns]: patterns.md#range-patterns
[Reference patterns]: patterns.md#reference-patterns
[Subpattern binding]: patterns.md#identifier-patterns
[Wildcard patterns]: patterns.md#wildcard-pattern
[arith]: expressions/operator-expr.md#arithmetic-and-logical-binary-operators
[array types]: types/array.md
[assignment]: expressions/operator-expr.md#assignment-expressions
[attributes]: attributes.md
[borrow]: expressions/operator-expr.md#borrow-operators
[closures]: expressions/closure-expr.md
[comparison]: expressions/operator-expr.md#comparison-operators
[compound]: expressions/operator-expr.md#compound-assignment-expressions
[constants]: items/constant-items.md
[dereference]: expressions/operator-expr.md#the-dereference-operator
[destructuring assignment]: expressions/underscore-expr.md
[extern crates]: items/extern-crates.md
[extern]: items/external-blocks.md
[field]: expressions/field-expr.md
[Floating-point literal expressions]: expressions/literal-expr.md#floating-point-literal-expressions
[floating-point types]: types/numeric.md#floating-point-types
[function pointer type]: types/function-pointer.md
[functions]: items/functions.md
[generics]: items/generics.md
[identifier]: identifiers.md
[if let]: expressions/if-expr.md#if-let-expressions
[Integer literal expressions]: expressions/literal-expr.md#integer-literal-expressions
[keywords]: keywords.md
[lazy-bool]: expressions/operator-expr.md#lazy-boolean-operators
[literal expressions]: expressions/literal-expr.md
[loop labels]: expressions/loop-expr.md
[macros]: macros-by-example.md
[match]: expressions/match-expr.md
[negation]: expressions/operator-expr.md#negation-operators
[negative impls]: items/implementations.md
[never type]: types/never.md
[numeric types]: types/numeric.md
[paths]: paths.md
[patterns]: patterns.md
[question]: expressions/operator-expr.md#the-question-mark-operator
[range]: expressions/range-expr.md
[rangepat]: patterns.md#range-patterns
[raw pointers]: types/pointer.md#raw-pointers-const-and-mut
[references]: types/pointer.md
[sized]: trait-bounds.md#sized
[String continuation escapes]: expressions/literal-expr.md#string-continuation-escapes
[struct expressions]: expressions/struct-expr.md
[trait bounds]: trait-bounds.md
[tuple index]: expressions/tuple-expr.md#tuple-indexing-expressions
[tuple structs]: items/structs.md
[tuple variants]: items/enumerations.md
[tuples]: types/tuple.md
[unary minus operator]: expressions/operator-expr.md#negation-operators
[use declarations]: items/use-declarations.md
[use wildcards]: items/use-declarations.md
[while let]: expressions/loop-expr.md#predicate-pattern-loops