# Literal expressions
# 字面量表达式

>[literal-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/literal-expr.md)\
>commit: 52e0ff3c11260fb86f19e564684c86560eab4ff9 \
>本章译文最后维护日期：2024-10-13

> **<sup>词法</sup>**\
> _LiteralExpression_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [CHAR_LITERAL]\
> &nbsp;&nbsp; | [STRING_LITERAL]\
> &nbsp;&nbsp; | [RAW_STRING_LITERAL]\
> &nbsp;&nbsp; | [BYTE_LITERAL]\
> &nbsp;&nbsp; | [BYTE_STRING_LITERAL]\
> &nbsp;&nbsp; | [RAW_BYTE_STRING_LITERAL]\
> &nbsp;&nbsp; | [C_STRING_LITERAL]\
> &nbsp;&nbsp; | [RAW_C_STRING_LITERAL]\
> &nbsp;&nbsp; | [INTEGER_LITERAL]\
> &nbsp;&nbsp; | [FLOAT_LITERAL]\
> &nbsp;&nbsp; | `true` | `false`>

字面表达式是由单一 token（而不是一组 token）组成的表达式，它立即和直接表示其表达式求值的值，而不是通过名称或其他求值规则来引用它。
它直接描述一个数字、字符、字符串或布尔值。

字面量是[常量表达式][constant expression]的一种形式，所以其求值（主要）在编译期。

前面描述的每种词法[字面量][literal tokens]形式都可以组成一个字面量表达式，比如关键字 `true` 和 `false` 也可以组成一个字面量表达式。

```rust
"hello";   // 字符串类型
'5';       // 字符类型
5;         // 整型
```

在下面的描述中，token 的 _字符串表示_ 是源自输入的字符序列，该字符序列会与*词法*语法片段中 token 的产生式进行模式匹配。

> **注意**：此字符串表示从不包括紧跟着 `U+000A` (LF) 的字符 `U+000D` (CR)：这一对字符会先被转换为单个字符 `U+000A` (LF)。

## Escapes
## 转义

下面对文本字面表达式的描述使用了几种形式的 _转义_。

每种形式的转义的特点是：
 * _转义序列_: 总是以 `U+005C` (`\`) 开头的一段字符序列
 * _转义值_: 单个字符或空的字符序列

在以下关于转义的定义中：
 * _八进制数字_ 是在 \[`0`-`7`] 区间内的任何字符。
 * _十六进制数字_ 是在 \[`0`-`9`]、\[`a`-`f`] 或 \[`A`-`F`] 这三个区间内的任何字符。

### Simple escapes
### 简单转义

下表第一列中出现的每个字符序列都是转义序列。

在每种情况下，转义值都是第二列中相应条目中给定的字符。

| 转义序列 | 转义值            |
|-----------------|--------------------------|
| `\0`            | U+0000 (NUL)             |
| `\t`            | U+0009 (HT)              |
| `\n`            | U+000A (LF)              |
| `\r`            | U+000D (CR)              |
| `\"`            | U+0022 (双引号)  |
| `\'`            | U+0027 (单引号)      |
| `\\`            | U+005C (反斜线) |

### 8-bit escapes
### 8bit字符值转义

这种转义序列由 `\x` 后跟两个十六进制数字组成。

转义值是一个字符，其 [Unicode标量值][Unicode scalar value]是将转义序列中的最后两个字符解释为十六进制整数的结果，就好像对这两个字符执行了以16为基数的[`u8::from_str_radix`]操作。

> **注意**: 作为结果的转义值因此具有在 [`u8`][numeric types] 数值区间内的 [Unicode标量值][Unicode scalar value]。

### 7-bit escapes
### 7bit字符值转义

转义序列由 `\x` 后跟一个八进制数字和一个十六进制数字组成。
转义值是一个字符，其[Unicode标量值][Unicode scalar value]是将转义序列中的最后两个字符解释为十六进制整数的结果，就好像对这两个字符执行了以16为基数的[`u8::from_str_radix`]操作。

### Unicode escapes
### Unicode字符值转义

转义序列由 `\u{` 后跟一系列字符组成，每个字符都是十六进制数字或 `_` ，在后跟一个 `}`。
转义值是一个字符，其[Unicode标量值][Unicode scalar value]是将转义序列中包含的十六进制数字解释为十六进制整数的结果，就好像对这一段字符执行了以16为基数的[`u32::from_str_radix`]操作。

> **注意**: [CHAR_LITERAL] token 或 [STRING_LITERA] token 的词法定义形式确保存在这样的字符。

### String continuation escapes
### 字符串接续符转义

转义序列由 `\` 后面紧跟着 `U+000A` (LF)，以及在下一个非空白符之前的所有后面的空白符组成。
为此，空白符被限定为 `U+0009` (HT)、`U+000A` (LF)、`U+000D` (CR) 和 `U+0020` (空格符)。

转义值是一个空的字符序列。

> **注意**: 这种转义形式的效果是字符串延续跳过后面的空白符（包括额外的换行符）。
> 因此下面 `a`、 `b` 和 `c` 是相等的:
> ```rust
> let a = "foobar";
> let b = "foo\
>          bar";
> let c = "foo\
>
>      bar";
>
> assert_eq!(a, b);
> assert_eq!(b, c);
> ```
>
> 跳过额外的换行符（如上面示例中的变量c 所示）可能会造成潜在混乱和意外。
> 这种行为将来可能会进行调整。
> 在做出决定之前，建议避免使用带有多个换行符接续符。
> 请参阅 [this issue](https://github.com/rust-lang/reference/pull/1042)，以了解更多信息。

## Character literal expressions
## 字符字面量表达式

字符字面量表达式由单一一个[字符字面量][CHAR_LITERAL]token 组成。

此表达式的类型是 rust的源语类型 [`char`][textual types]。

此类token 不能带有后缀。

此类token 的 _字面内容_ 是在此token的字符串表示中第一个 `U+0027` (`'`) 之后，最后一个 `U+0027` (`'`) 之前的字符序列。

此类字面量表达式的 _所表示字符_ 源自字面内容，具体有如下规则：
* 如果字面内容是以下转义序列形式之一，则所表示的字符是转义序列的转义值：
    * [简单转义][Simple escapes]
    * [7bit转义][7-bit escapes]
    * [Unicode转义][Unicode escapes]
* 否则，所表示的字符是构成字面内容的单个字符。

表达式的值是与所表示的字符的[Unicode标量值][Unicode scalar value]相对应的 [`char`][text types]类型。

> **注意**: [CHAR_LITERAL] token 的词法定义形式确保会产生单一字符。

字符字面量表达式示例：

```rust
'R';                               // R
'\'';                              // '
'\x52';                            // R
'\u{00E6}';                        // 拉丁文小写字母æ (U+00E6)
```

## String literal expressions
## 字符串字面量表达式

字符串字面量表达式由单一一个[字符串字面量][STRING_LITERAL]token 或[原生字符串字面量][RAW_STRING_LITERAL]token 组成。

此类表达式的类型是对原语类型 [`str`][text types] 的共享引用（带有 `static`生存期)。
也就是说，类型是 `&'static str`。

此类token 不能带有后缀。

此类token 的 _字面内容_ 是此token 的字符串表示中第一个 `U+0022` (`"`) 之后和最后一个 `U+0022` (`"`) 之前的字符序列。

此字面量表达式的 _所表示字符串_ 源自于字符序列的字面内容，具体有如下规则：

* 如果 token 是[STRING_LTERAL]词法所限定的，则字面内容中出现的以下任何形式的转义序列都将被转义序列的转义值替换。
    * [简单转义][Simple escapes]
    * [7bit转义][7-bit escapes]
    * [Unicode转义][Unicode escapes]
    * [字符串接续符转义][String continuation escapes]

  这些转义替换按从左到右的顺序进行的。
  例如，token `"\\x41"` 被转换为字符 `\` `x` `4` `1`。

* 如果 token 是[RAW_STRING_LITERAL]词法所限定的，则所表示的字符串等同于字面内容。

表达式的值是对静态分配的 [`str`][text types]的引用，该引用包含所表示字符串的 UTF-8编码形式。

字符串字面量表达式示例：

```rust
"foo"; r"foo";                     // foo
"\"foo\""; r#""foo""#;             // "foo"

"foo #\"# bar";
r##"foo #"# bar"##;                // foo #"# bar

"\x52"; "R"; r"R";                 // R
"\\x52"; r"\x52";                  // \x52
```

## Byte literal expressions
## 字节字面量表达式

字节字面量表达式由单一一个[字节字面量][BYTE_LITERAL]token 组成。

此表达式的类型是 rust的源语类型 [`u8`][numeric types]。

此类token 不能带有后缀。

此类token 的 _字面内容_ 是此token 的字符串表示中第一个 `U+0027` (`'`) 之后和最后一个 `U+0027` (`'`) 之前的字符序列。

此字面量表达式的 _所表示字符_ 源自于字符的字面内容，具体有如下规则：
* 如果字面内容是以下转义序列形式之一，则表示的字符是转义序列的转义值：
    * [简单转义][Simple escapes]
    * [8bit转义][8-bit escapes]

* 否则，所表示的字符是构成字面内容的单个字符。

此类表达式的值是所表示的字符的[Unicode标量值][Unicode scalar value]。

> **注意**: [BYTE_LTERAL] token 的词法定义形式确保这些规则始终能生成一个 Unicode标量值在 [`u8`][numeric types] 数值区间内的单个字符。

字节字面量表达式的示例：

```rust
b'R';                              // 82
b'\'';                             // 39
b'\x52';                           // 82
b'\xA0';                           // 160
```

## Byte string literal expressions
## 字节串字面量表达式

字节串字面量表达式由单一一个[字节串字面量][BYTE_STRING_LITERAL]token 或[原生字节串字面量][RAW_BYTE_STRING_LITERAL]token 组成。

此类表达式的类型是对元素类型为 [`u8`][numeric-types]的数组的共享引用（带有 `static`生存期）。
也就是说，类型是 `&'static [u8; N]`，其中 `N` 是下文描述的所表示的字符串的字节数。

此类token 不能带有后缀。

此类token 的 _字面内容_ 是此token 的字符串表示中第一个 `U+0022` (`"`) 之后和最后一个 `U+0022` (`"`) 之前的字符序列。

此字面量表达式的 _所表示字符串_ 源自于字符序列的字面内容，具体有如下规则：

* 如果 token 是[BYTE_STRING_LITERAL]词法所限定的，则字面内容中出现的以下任何形式的每个转义序列都将替换为转义序列的转义值。
    * [简单转义][Simple escapes]
    * [8bit转义][8-bit escapes]
    * [字符串接续符转义][String continuation escapes]

  这些转义替换按从左到右的顺序进行的。
  例如，token `b"\\x41"` 被转换为字符 `\` `x` `4` `1`。

* 如果 token 是[RAW_BYTE_STRING_LITERAL]词法所限定的，则所表示的字符串等同于字面内容。

表达式的值是对静态分配的数组的引用，该数组包含所表示字符串中字符的[Unicode标量值][Unicode scalar values]，顺序相同。

> **注意**: [BYTE_STRING_LITERAL] 和 [RAW_BYTE_STRING_LITERAL] 的词法定义形式确保这些规则生成数组元素值始终在 [`u8`][numeric types] 数值区间内。

字节串字面量表达式的示例:

```rust
b"foo"; br"foo";                     // foo
b"\"foo\""; br#""foo""#;             // "foo"

b"foo #\"# bar";
br##"foo #"# bar"##;                 // foo #"# bar

b"\x52"; b"R"; br"R";                // R
b"\\x52"; br"\x52";                  // \x52
```

## C string literal expressions
## C语言风格的字符串字面量表达式

C语言风格的字符串字面量表达式由单一一个[C语言风格的字符串字面量][C_STRING_LITERAL] 或 [原生C语言风格的字符串字面量][RAW_C_STRING_LITERAL] token 组成。

此类表达式的类型是对标准库[CStr]类型的共享引用（带有 `static`生存期）。
也就是说，类型是 `&'static core::ffi::CStr`。

此类token 不能带有后缀。

此类token 的 _字面内容_ 是此token 的字符串表示中第一个 `U+0022` (`"`) 之后和最后一个 `U+0022` (`"`) 之前的字符序列。

此类字面量表达式的 _所表示字节序_ 源自于字节序列的字面内容，具体有如下规则：

* 如果 token 是[C_STRING_LITERAL]词法所限定的，则字面内容被视为一系列内容项（下面有罗列），每个内容项要么是除 `\` 之外的单个 Unicode字符，要么是[转义][escape]。
内容项序列会被转换为字节序列，具体规则如下所示：
  * 每个 Unicode字符都有 UTF-8表示形式。
  * 每个[简单转义][simple escape]都会被转义为其转义值的[Unicode标量值][Unicode scalar value]。
  * 每个[8bit转义][8-bit escape]都会被转义为一个包含其转义值的[Unicode标量值][Unicode scalar value]的单个字节。
  * 每个[Unicode转义][unicode escape]都会被转义为其转义值的 UTF-8表示形式。
  * 每个[字符串接续符转义][string continuation escape]都不会被转义为任何字节。

* 如果 token [RAW_C_STRING_LITERAL] 词法所限定的，则表示的字节是字面内容的 UTF-8编码。

> **注意**: [C_STRING_LITERAL] 和 [RAW_C_STRING_LITERAL]的词法定义形式确保所表示的字节序不会包含 null字节。

The expression's value is a reference to a statically allocated [CStr] whose array of bytes contains the represented bytes followed by a null byte.
该表达式的值是对静态分配的 [CStr] 的引用，其字节数组就包含了这个后跟一个 null字节的所表示字节序。

C语言风格的字符串字面量表达式示例:

```rust
c"foo"; cr"foo";                     // foo
c"\"foo\""; cr#""foo""#;             // "foo"

c"foo #\"# bar";
cr##"foo #"# bar"##;                 // foo #"# bar

c"\x52"; c"R"; cr"R";                // R
c"\\x52"; cr"\x52";                  // \x52

c"æ";                                // 拉丁文小写字母æ (U+00E6)
c"\u{00E6}";                         // 拉丁文小写字母æ (U+00E6)
c"\xC3\xA6";                         // 拉丁文小写字母æ (U+00E6)

c"\xE6".to_bytes();                  // [230]
c"\u{00E6}".to_bytes();              // [195, 166]
```

## Integer literal expressions
## 整型字面量表达式

整型字面量表达式由单一一个[整型字面量][INTEGER_LITERAL]token 组成。

如果这种 token 带有[后缀][suffix]，那这个后缀的名字必须是[原生整型][numeric types]中的一个：`u8`, `i8`, `u16`, `i16`, `u32`, `i32`, `u64`, `i64`, `u128`, `i128`, `usize` 或 `isize`，同时此后缀名也是此表达式的类型。

如果此 token 没有后缀，此表达式的类型将由下面的类型推断规则来确定:

* 如果整型可以从周围的程序上下文中*唯一地*确定，则表达式具有该类型。

* 如果程序上下文对类型的约束比较宽松，则默认为有符号32位整数 `i32`。

* 如果程序上下文对类型的约束过度，则被认为是一个静态类型错误。

整型字面量表达式的示例：

```rust
123;                               // type i32
123i32;                            // type i32
123u32;                            // type u32
123_u32;                           // type u32
let a: u64 = 123;                  // type u64

0xff;                              // type i32
0xff_u8;                           // type u8

0o70;                              // type i32
0o70_i16;                          // type i16

0b1111_1111_1001_0000;             // type i32
0b1111_1111_1001_0000i64;          // type i64

0usize;                            // type usize
```

而表达式的值由 token 的字符串表示形式来确定，规则如下：

* 通过检查字符串的前两个字符来选择整型的进制数，如下所示：

    * `0b` 表示进制数为2
    * `0o` 表示进制数为8
    * `0x` 表示进制数为16
    * 其他表示进制数为10。

* 如果进制数不是10，则从字符串中删除前两个字符。

* 字符串的后缀都会被删除。

* 字符串里的下划线都会被删除。

* 字符串被转换为 `u128`类型的值时，比如使用 [`u128::from_str_radix`] 时选的进制数标记。
如果该值不能被表示为 `u128`，将在编译时报错。

* `u128`类型的值通过[数值转换][numeric cast]转换为表达式的类型。

> **注意**: 如果字面量的值不适合表达式的类型，则会对最终强制转换的结果做截断操作。
> `rustc` 包含一个名为 `overflowing_literals` 的 [lint检查][lint check]， 默认生效级别为 `deny`， 这意味着这种情况发生是此表达式会被编译器拒绝。

> **注意**： `-1i8` 这样的表达式是对 `1i8` 这个字面量表达式的[取负操作][negation operator]，不是一个单一的整型字面量表达式。
> 请参阅[Overflow]，了解有关表示有符号类型的最大负值的说明。
## Floating-point literal expressions

## 浮点型字面量表达式

浮点型字面量表达式由下面两种形式：[浮点型字面量][FLOAT_LITERAL]token 组成。
 * 由单一的[浮点型字面量][FLOAT_LITERAL]token 组成
 * 由单一的没有后缀和基数指示符的[整型字面量][INTEGER_LITERAL]token 组成

如果这种 token 带有[后缀][suffix]，那这个后缀的名字必须是[原生浮点型][floating-point types]中的一个：`f32` 或 `f64`，同时此后缀名也是此表达式的类型。

如果这种 token 没有后缀，此表达式的类型将由下面的类型推断规则来确定:

* 如果浮点型可以从周围的程序上下文中*唯一地*确定，则表达式具有该类型。

* 如果程序上下文对类型的约束比较宽松，则默认为 `f64`。

* 如果程序上下文对类型的约束过度，则被认为是一个静态类型错误。

浮点型字面量表达式的示例：

```rust
123.0f64;        // type f64
0.1f64;          // type f64
0.1f32;          // type f32
12E+99_f64;      // type f64
5f32;            // type f32
let x: f64 = 2.; // type f64
```

而表达式的值由 token 的字符串表示形式来确定，规则如下：

* 字符串的后缀都会被删除。

* 字符串中的下划线都会被删除。

* 字符串被转换为这类表达式类型时，如同直接调用 [`f32::from_str`] 或 [`f64::from_str`]。

> **注意**: `-1.0` 这样的表达式是对 `1.0` 这个字面量表达式的[取负操作][negation operator]，不是一个单一的浮点型字面量表达式。

> **注意**: `inf` 和 `NaN` 不是字面量token。
> [`f32::INFINITY`], [`f64::INFINITY`], [`f32::NAN`] 和 [`f64::NAN`] 这些常量可被用来替代字面量表达式。

> 在 `rustc` 里，一个足够大的字面量被计算为无穷时将会触发一个叫做 `overflowing_literals` 的 lint检查。

## Boolean literal expressions
## 布尔型字面量表达式

布尔型字面量表达式由关键字 `true` 或 `false` 中的一个组成。

此类表达式的类型是原生[布尔型][boolean type]，并且它的值为：
 * 如果关键字是 `true`，那么值为 true
 * 如果关键字是 `false`，那么值为 false
[Escape]: #escapes
[Simple escape]: #simple-escapes
[Simple escapes]: #simple-escapes
[8-bit escape]: #8-bit-escapes
[8-bit escapes]: #8-bit-escapes
[7-bit escape]: #7-bit-escapes
[7-bit escapes]: #7-bit-escapes
[Unicode escape]: #unicode-escapes
[Unicode escapes]: #unicode-escapes
[String continuation escape]: #string-continuation-escapes
[String continuation escapes]: #string-continuation-escapes
[boolean type]: ../types/boolean.md
[constant expression]: ../const_eval.md#constant-expressions
[CStr]: core::ffi::CStr
[floating-point types]: ../types/numeric.md#floating-point-types
[lint check]: ../attributes/diagnostics.md#lint-check-attributes
[literal tokens]: ../tokens.md#literals
[numeric cast]: operator-expr.md#numeric-cast
[numeric types]: ../types/numeric.md
[suffix]: ../tokens.md#suffixes
[negation operator]: operator-expr.md#negation-operators
[overflow]: operator-expr.md#overflow
[textual types]: ../types/textual.md
[Unicode scalar value]: http://www.unicode.org/glossary/#unicode_scalar_value
[Unicode scalar values]: http://www.unicode.org/glossary/#unicode_scalar_value
[`f32::from_str`]: https://doc.rust-lang.org/core/primitive.f32.html#method.from_str
[`f64::from_str`]: https://doc.rust-lang.org/core/primitive.f64.html#method.from_str
[CHAR_LITERAL]: ../tokens.md#character-literals
[STRING_LITERAL]: ../tokens.md#string-literals
[RAW_STRING_LITERAL]: ../tokens.md#raw-string-literals
[BYTE_LITERAL]: ../tokens.md#byte-literals
[BYTE_STRING_LITERAL]: ../tokens.md#byte-string-literals
[RAW_BYTE_STRING_LITERAL]: ../tokens.md#raw-byte-string-literals
[C_STRING_LITERAL]: ../tokens.md#c-string-literals
[RAW_C_STRING_LITERAL]: ../tokens.md#raw-c-string-literals
[INTEGER_LITERAL]: ../tokens.md#integer-literals
[FLOAT_LITERAL]: ../tokens.md#floating-point-literals