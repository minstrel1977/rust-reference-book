# Literal expressions
# 字面量表达式

>[literal-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/literal-expr.md)\
>commit: ccde77edcb4d4607f212bfd9fa65a913defb5055 \
>本章译文最后维护日期：2022-12-04

> **<sup>句法</sup>**\
> _LiteralExpression_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [CHAR_LITERAL]\
> &nbsp;&nbsp; | [STRING_LITERAL]\
> &nbsp;&nbsp; | [RAW_STRING_LITERAL]\
> &nbsp;&nbsp; | [BYTE_LITERAL]\
> &nbsp;&nbsp; | [BYTE_STRING_LITERAL]\
> &nbsp;&nbsp; | [RAW_BYTE_STRING_LITERAL]\
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
## Character literal expressions
## 字符字面量表达式

字符字面量表达式由单一一个[字符字面量][CHAR_LITERAL]token 组成。

> **注意**: 本节还未编写完成。

## String literal expressions
## 字符串字面量表达式

字符串字面量表达式由单一一个[字符串字面量][STRING_LITERAL]token 或[裸字符串字面量][RAW_STRING_LITERAL]token 组成。

> **注意**: 本节还未编写完成。

## Byte literal expressions
## 字节字面量表达式

字节字面量表达式由单一一个[字节字面量][BYTE_LITERAL]token 组成。

> **注意**: 本节还未编写完成。

## Byte string literal expressions
## 字节串字面量表达式

字节串字面量表达式由单一一个[字节串字面量][BYTE_STRING_LITERAL]token 或[裸字节串字面量][RAW_BYTE_STRING_LITERAL]token 组成。

> **注意**: 本节还未编写完成。

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

[boolean type]: ../types/boolean.md
[constant expression]: ../const_eval.md#constant-expressions
[floating-point types]: ../types/numeric.md#floating-point-types
[lint check]: ../attributes/diagnostics.md#lint-check-attributes
[literal tokens]: ../tokens.md#literals
[numeric cast]: operator-expr.md#numeric-cast
[numeric types]: ../types/numeric.md
[suffix]: ../tokens.md#suffixes
[negation operator]: operator-expr.md#negation-operators
[overflow]: operator-expr.md#overflow
[`f32::from_str`]: https://doc.rust-lang.org/core/primitive.f32.html#method.from_str
[`f32::INFINITY`]: https://doc.rust-lang.org/core/primitive.f32.md#associatedconstant.INFINITY
[`f32::NAN`]: https://doc.rust-lang.org/core/primitive.f32.md#associatedconstant.NAN
[`f64::from_str`]: https://doc.rust-lang.org/core/primitive.f64.md#method.from_str
[`f64::INFINITY`]: https://doc.rust-lang.org/core/primitive.f64.md#associatedconstant.INFINITY
[`f64::NAN`]: https://doc.rust-lang.org/core/primitive.f64.md#associatedconstant.NAN
[`u128::from_str_radix`]: https://doc.rust-lang.org/core/primitive.u128.md#method.from_str_radix
[CHAR_LITERAL]: ../tokens.md#character-literals
[STRING_LITERAL]: ../tokens.md#string-literals
[RAW_STRING_LITERAL]: ../tokens.md#raw-string-literals
[BYTE_LITERAL]: ../tokens.md#byte-literals
[BYTE_STRING_LITERAL]: ../tokens.md#byte-string-literals
[RAW_BYTE_STRING_LITERAL]: ../tokens.md#raw-byte-string-literals
[INTEGER_LITERAL]: ../tokens.md#integer-literals
[FLOAT_LITERAL]: ../tokens.md#floating-point-literals