# 操作符/运算符表达式
# Operator expressions

>[operator-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/operator-expr.md)\
>commit: 03dc50769738a643be1451a4ff1516fa5fab92bd \
>本译文最后维护日期：2020-10-26

> **<sup>句法</sup>**\
> _OperatorExpression_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_BorrowExpression_]\
> &nbsp;&nbsp; | [_DereferenceExpression_]\
> &nbsp;&nbsp; | [_ErrorPropagationExpression_]\
> &nbsp;&nbsp; | [_NegationExpression_]\
> &nbsp;&nbsp; | [_ArithmeticOrLogicalExpression_]\
> &nbsp;&nbsp; | [_ComparisonExpression_]\
> &nbsp;&nbsp; | [_LazyBooleanExpression_]\
> &nbsp;&nbsp; | [_TypeCastExpression_]\
> &nbsp;&nbsp; | [_AssignmentExpression_]\
> &nbsp;&nbsp; | [_CompoundAssignmentExpression_]

操作符是Rust 语言为其内建类型定义的。本文后面的许多操作符都可以使用 `std::ops` 或 `std::cmp` 中的 trait 进行重载。

## 溢出
## Overflow

在调试模式下编译整数运算时，如果发生溢出，会触发 panic。可以使用命令行参数 `-C debug-assertions` 和 `-C overflow-checks` 来设置编译器标志位，来更直接地控制这个溢出过程。以下情况被认为是溢出：

* 当 `+`、`*` 或 `-` 创建的值大于当前类型可存储的最大值或小于最小值。这包括任何有符号整型的最小值上的一元运算符 `-`。
* 使用 `/` 或 `%`，其中左边的参数是某类有符号整型的最小整数，右边的参数是 `-1`。
* 使用 `<<` 或 `>>`，其中右边参数大于或等于左边参数类型的比特数，或右边参数为负数。

## 借用/引用操作符
## Borrow operators

> **<sup>句法</sup>**\
> _BorrowExpression_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; (`&`|`&&`) [_Expression_]\
> &nbsp;&nbsp; | (`&`|`&&`) `mut` [_Expression_]

`&`（共享借用）和 `&mut`（可变借用）运算符是一元前缀运算符。当应用于[位置表达式][place expression]上时，此表达式生成指向值所在的内存位置的引用（指针）。在引用期间，该内存位置也被置于借出状态。对于共享借用（`&`），这意味着该位置可能不会发生变化，但可能会被再次读取或共享。对于可变借用（`&mut`），在借用到期之前，不能以任何方式访问该位置。`&mut` 在可变位置表达式上下文中计算其操作数。如果 `&` 或 `&mut` 运算符应用于[值表达式][value expression]，则会创建一个[临时值][temporary value]。

这类操作符不能重载。

```rust
{
    // 将创建一个存值为7的临时位置，该该临时位置在此作用域内持续存在
    let shared_reference = &7;
}
let mut array = [-2, 3, 9];
{
    // 在当前作用域内可变借用了 `array`。那 `array` 就只能通过 `mutable_reference` 来使用。
    let mutable_reference = &mut array;
}
```

尽管 `&&` 是一个单一 token（[惰性与(`and`)操作符](#lazy-boolean-operators))，但在借用表达式(borrow expressions)上下文中使用时，它是作为两个借用操作符用的：

```rust
// 意义相同：
let a = &&  10;
let a = & & 10;

// 意义相同：
let a = &&&&  mut 10;
let a = && && mut 10;
let a = & & & & mut 10;
```

## 解引用操作符
## The dereference operator

> **<sup>句法</sup>**\
> _DereferenceExpression_ :\
> &nbsp;&nbsp; `*` [_Expression_]

`*`（解引用）操作符也是一元前缀操作符。当应用于[指针][pointer]时，它表示该指针指向的内存位置。如果表达式的类型为 `&mut T` 或 `*mut T`，并且该表达式是局部变量（局部变量的（嵌套）字段也可以）或是可变的[位置表达式][place expression]，则它代表的内存位置可以被赋值。解引用原始指针需要在非安全(`unsafe`)块才能进行。

在[不可变位置表达式上下文][immutable place expression context]中对非指针类型作 `*x` 相当于执行 `*std::ops::Deref::deref(&x)`；同样的，在可变位置表达式上下文中这个动作就相当于执行 `*std::ops::DerefMut::deref_mut(&mut x)`。

```rust
let x = &7;
assert_eq!(*x, 7);
let y = &mut 9;
*y = 11;
assert_eq!(*y, 11);
```

## 问号操作符
## The question mark operator

> **<sup>句法</sup>**\
> _ErrorPropagationExpression_ :\
> &nbsp;&nbsp; [_Expression_] `?`

问号操作符（`?`）解开(unwrap)有效值或返回错误值，并将它们传播(propagate)给调用函数。问号操作符（`?`）是一个一元后缀操作符，只能应用于类型 `Result<T, E>` 和 `Option<T>`。

当应用在 `Result<T, E>` 类型的值上时，它可以传播错误。如果值是 `Err(e)`，那么它将实际从函数体或闭包中返回 `Err(From::from(e))`。如果应用到 `Ok(x)`，那么它将展开值以求得 `x`。

```rust
# use std::num::ParseIntError;
fn try_to_parse() -> Result<i32, ParseIntError> {
    let x: i32 = "123".parse()?; // x = 123
    let y: i32 = "24a".parse()?; // 立即返回一个 Err()
    Ok(x + y)                    // 不会执行到这里
}

let res = try_to_parse();
println!("{:?}", res);
# assert!(res.is_err())
```

当应用到 `Option<T>` 类型的值时，它向调用者传播错误 `None`。如果它应用的值本身是 `None`，那么它将返回 `None`。如果应用于 `Some(x)`，那么它将解开值以求得 `x`。

```rust
fn try_option_some() -> Option<u8> {
    let val = Some(1)?;
    Some(val)
}
assert_eq!(try_option_some(), Some(1));

fn try_option_none() -> Option<u8> {
    let val = None?;
    Some(val)
}
assert_eq!(try_option_none(), None);
```

操作符 `?` 不能被重载。

## 取反运算符
## Negation operators

> **<sup>语法</sup>**\
> _NegationExpression_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `-` [_Expression_]\
> &nbsp;&nbsp; | `!` [_Expression_]

这是最后两个一元运算符。下表总结了它们用在基本类型上的表现，以及指出其他类型要重载这些操作符需要实现哪些 trait。记住，有符号整数总是用2的补码形式表示。所有这些运算符的操作数都在[值表达式上下文][value expression]中被求值，所以这些操作数的值会被移走或复制。

| 符号 | 整数     | `bool`      | 浮点数 | 用于重载的 trait  |
|--------|-------------|-------------|----------------|--------------------|
| `-`    | 符号取反*   |             | 符号取反       | `std::ops::Neg`    |
| `!`    | 按位取反 | 逻辑非 |                | `std::ops::Not`    |

\* 仅适用于有符号整数类型。

下面是这些运算符的一些示例：

```rust
let x = 6;
assert_eq!(-x, -6);
assert_eq!(!x, -7);
assert_eq!(true, !false);
```

## 算术和逻辑二元运算符
## Arithmetic and Logical Binary Operators

> **<sup>句法</sup>**\
> _ArithmeticOrLogicalExpression_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_Expression_] `+` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `-` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `*` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `/` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `%` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `&` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `|` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `^` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `<<` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `>>` [_Expression_]

二元运算符表达式都用中缀表示法(infix notation)书写。下表总结了算术和逻辑二元运算符在原生类型(primitive type)上的行为，以及指出其他类型要重载这些操作符需要实现哪些 trait。记住，有符号整数总是用2的补码形式表示。所有这些运算符的操作数都在[值表达式上下文][value expression]中求值，因此这些操作数的值会被移走或复制。

| 符号 | 整数                 | `bool`      | 浮点数 | 用于重载的 trait  |
|--------|-------------------------|-------------|----------------|--------------------|
| `+`    | 加法                |             | 加法       | `std::ops::Add`    |
| `-`    | 减法             |             | 减法    | `std::ops::Sub`    |
| `*`    | 乘法          |             | 乘法 | `std::ops::Mul`    |
| `/`    | 除法*                |             | 取余       | `std::ops::Div`    |
| `%`    | 取余               |             | Remainder      | `std::ops::Rem`    |
| `&`    | 按位与             | 逻辑与 |                | `std::ops::BitAnd` |
| <code>&#124;</code> | 按位或 | 逻辑或  |                | `std::ops::BitOr`  |
| `^`    | 按位异或             | 逻辑异或 |                | `std::ops::BitXor` |
| `<<`   | 左移位              |             |                | `std::ops::Shl`    |
| `>>`   | 右移位**            |             |                | `std::ops::Shr`    |

\* 整数除法趋零取整。

\*\* 有符号整数类型算术右移位，无符号整数类型逻辑右移位。

下面是使用这些操作符的示例:

```rust
assert_eq!(3 + 6, 9);
assert_eq!(5.5 - 1.25, 4.25);
assert_eq!(-5 * 14, -70);
assert_eq!(14 / 3, 4);
assert_eq!(100 % 7, 2);
assert_eq!(0b1010 & 0b1100, 0b1000);
assert_eq!(0b1010 | 0b1100, 0b1110);
assert_eq!(0b1010 ^ 0b1100, 0b110);
assert_eq!(13 << 3, 104);
assert_eq!(-10 >> 2, -3);
```

## 比较运算符
## Comparison Operators

> **<sup>句法</sup>**\
> _ComparisonExpression_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_Expression_] `==` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `!=` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `>` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `<` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `>=` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `<=` [_Expression_]

Rust 还为原生类型以及标准库中的许多类型定义了比较运算符。链式比较运算时需要括号，例如，表达式 `a == b == c` 是无效的，（但如果逻辑允许）可以写成 `(a == b) == c`。

与算术运算符和逻辑运算符不同，比较运算符所使用的 trait（这些 trait 也可以被其他类型所实现以重载这些比较运算符）更抽象地用于显示/约定如何比较一个类型，并且很可能会假定使用这些 trait 作为约束条件的函数定义了实际的比较逻辑。（对库API使用者来说，）标准库中的许多函数和宏都可以被认为遵守了这个假定(尽管不能确保它们的实现的安全性)。与上面的算术和逻辑运算符不同，这些运算符会隐式地对它们的操作数执行共享借用，并当在[位置表达式上下文][place expression]中对它们进行求值：

```rust
# let a = 1;
# let b = 1;
a == b;
// 等价于：
::std::cmp::PartialEq::eq(&a, &b);
```

这意味着不需要将值从操作数移出(moved out of)。

| 符号 | 含义                  | 须重载方法         |
|--------|--------------------------|----------------------------|
| `==`   | 等于                    | `std::cmp::PartialEq::eq`  |
| `!=`   | 不等于                | `std::cmp::PartialEq::ne`  |
| `>`    | 大于             | `std::cmp::PartialOrd::gt` |
| `<`    | 小于                | `std::cmp::PartialOrd::lt` |
| `>=`   | 大于或等于 | `std::cmp::PartialOrd::ge` |
| `<=`   | 小于或等于    | `std::cmp::PartialOrd::le` |

下面是使用比较运算符的示例：

```rust
assert!(123 == 123);
assert!(23 != -12);
assert!(12.5 > 12.2);
assert!([1, 2, 3] < [1, 3, 4]);
assert!('A' <= 'B');
assert!("World" >= "Hello");
```

## 短路布尔运算符
## Lazy boolean operators

> **<sup>句法</sup>**\
> _LazyBooleanExpression_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_Expression_] `||` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `&&` [_Expression_]

运算符 `||` 和 `&&` 可以应用在布尔类型的操作数上。运算符 `||` 表示逻辑“或”，运算符 `&&` 表示逻辑“与”。它们与 `|` 和 `&` 的不同之处在于，只有在左操作数尚未确定表达式的结果时，才计算右操作数。也就是说，`||` 只在左操作数的计算结果为 `false` 时才计算其右操作数，而只有在计算结果为 `true` 时才计算 `&&` 的操作数。

```rust
let x = false || true; // true
let y = false && panic!(); // false, 不会计算 `panic!()`
```

## 类型转换表达式
## Type cast expressions

> **<sup>句法</sup>**\
> _TypeCastExpression_ :\
> &nbsp;&nbsp; [_Expression_] `as` [_TypeNoBounds_]

类型转换表达式用二元运算符 `as` 表示。

执行 `as`表达式将左侧的值显式转换为右侧的类型。

`as`表达式的一个例子：

```rust
# fn sum(values: &[f64]) -> f64 { 0.0 }
# fn len(values: &[f64]) -> i32 { 0 }
fn average(values: &[f64]) -> f64 {
    let sum: f64 = sum(values);
    let size: f64 = len(values) as f64;
    sum / size
}
```

`as` 可用于显式执行[自动强转(coercions)][coercions]，以及下列形式的强制转换。下表中 `*T` 代表 `*const T` 或 `*mut T`。

| `e` 的类型          | `U`                   | 通过 `e as U` 执行转换      |
|-----------------------|-----------------------|----------------------------------|
| 整型或浮点型           | 整型或浮点型           | 数字转换                     |
| 类C(C-like)枚举        | 整型          | 枚举转换                        |
| `bool` 或 `char`      | 整型          | 原生类型到整型的转换        |
| `u8`                  | `char`                | `u8` 到 `char` 的转换              |
| `*T`                  | `*V` where `V: Sized` \* | 指针到指针的转换       |
| `*T` where `T: Sized` | 数字型(Numeric type)         |  指针到地址的转换         |
| 整型          | `*V` where `V: Sized` | 地址到指针的转换          |
| `&[T; n]`             | `*const T`            | 数组到指针的转换            |
| [函数项][Function item]       | [函数指针][Function pointer]    | 函数到函数指针的转换 |
| [函数项][Function item]       | `*V` where `V: Sized` | 函数到指针的转换    |
| [函数项][Function item]       | 整型               | 函数到地址的转换    |
| [函数指针][Function pointer]    | `*V` where `V: Sized` | 函数指针到指针的转换  |
| [函数指针][Function pointer]    | 整型               | 函数指针到地址的转换 |
| 闭包 \*\*          | 函数指针      | 闭包到函数指针的转换 |

\* 或者 `T`和`V` 也可以都是兼容的 unsized 类型，例如，两个都是切片，或者都是同一种 trait对象。

\*\* 仅适用于不捕获（遮蔽(close over)）任何环境变量的闭包。

### 语义
### Semantics

* 数字转换(Numeric cast)
    * 在两个尺寸(size)相同的整型数值（例如 i32 -> u32）之间进行转换是一个空操作(no-op)
    * 从一个较大尺寸的整型转换为较小尺寸的整型（例如 u32 -> u8）将会采用截断(truncate)算法 [^译者注]
    * 从较小尺寸的整型转换为较大尺寸的整型（例如 u8 -> u32）将
        * 如果源数据是无符号的，则进行零扩展(zero-extend)
        * 如果源数据是有符号的，则进行符号扩展(sign-extend)
    * 从浮点数转换为整型将使浮点数趋零取整(round the float towards zero)
        * `NaN` 将返回 `0`
        * 大于转换到的整型类型的最大值时，取该整型类型的最大值。
        * 小于转换到的整型类型的最小值时，取该整型类型的最小值。
    * 从整数强制转换为浮点数将产生最接近的浮点数 \*
        * 如有必要，舍入采用 `roundTiesToEven` 模式 \*\*\*
        * 在溢出时，将会产生该浮点型的常量 Infinity(∞)（与输入符号相同）
        * 注意：对于当前的数值类型集，溢出只会发生在 `u128 as f32` 这种转换形式，且数字大于或等于 `f32::MAX + (0.5 ULP)` 时。
    * 从 f32 到 f64 的转换是无损转换
    * 从 f64 到 f32 的转换将产生最接近的 f32 \*\*
        * 如有必要，舍入采用 `roundTiesToEven` 模式 \*\*\*
        * 在溢出时，将会产生 f32 的常量 Infinity(∞)（与输入符号相同）
* 枚举转换(Enum cast)
    * 先将枚举转换为它的判别值(discriminant)，然后在需要时使用数值转换。
* 原生类型到整型的转换
    * `false` 转换为 `0`, `true` 转换为 `1`
    * `char` 会先强制转换为代码点的值，然后在需要时使用数值转换。
* `u8` 到 `char` 的转换
    * 转换为具有相应代码点的 `char` 值。

\* 如果硬件本身不支持这种舍入模式和溢出行为，那么这些整数到浮点型的转换可能会比预期的要慢。

\*\* 如果硬件本身不支持这种舍入模式和溢出行为，那么这些 f64 到 f32 的转换可能会比预期的要慢。

\*\*\* 按照 IEEE 754-2008§4.3.1 的定义：选择最接近的浮点数，如果恰好在两个浮点数中间，则优先选择最低有效位为偶数的那个。

## 赋值表达式
## Assignment expressions

> **<sup>句法</sup>**\
> _AssignmentExpression_ :\
> &nbsp;&nbsp; [_Expression_] `=` [_Expression_]

*赋值表达式*由[位置表达式][place expression]后跟等号（`=`）和[值表达式][value expression]组成。这样的表达式的类型总是[单元(`unit`)类型][`unit` type]。

执行赋值表达式时会先[销毁(drop)][drops]左操作数（如果是未初始化的局部变量或未初始化的局部变量的字段则不会启动这步析构操作），然后将其右操作数[复制(copy)或移动(move)][either copies or moves]到左操作数。左边的操作数必须是位置表达式：使用值表达式不会将其提升为临时位置，而是会导致编译器错误。

```rust
# let mut x = 0;
# let y = 0;
x = y;
```

## 复合赋值表达式
## Compound assignment expressions

> **<sup>句法</sup>**\
> _CompoundAssignmentExpression_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_Expression_] `+=` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `-=` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `*=` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `/=` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `%=` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `&=` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `|=` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `^=` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `<<=` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `>>=` [_Expression_]

`+`, `-`, `*`, `/`, `%`, `&`, `|`, `^`, `<<`, 和 `>>`运算符可以和 `=` 运算符复合组成新的运算符。表达式 `place_exp OP= value` 等效于 `place_expr = place_expr OP val`。例如，`x = x + 1` 可以写成 `x += 1`。任何这样的表达式的类型总是[单元(`unit`)类型][`unit` type]。这些运算符都可以使用与普通操作的相同的 trait 来重载，只是相应的 trait名称后要加上 “Assign”，例如，`std::ops::AddAssign` 用于重载 `+=`。因为带有 `=`，所以 `place_expr` 必须是[位置表达式][place expression]。

```rust
let mut x = 10;
x += 4;
assert_eq!(x, 14);
```
[^译者注]:截断，即一个值范围较大的变量A转换为值范围较小的变量B，如果超出范围，则将A减去B的区间长度。例如，128超出了i8类型的范围（-128,127），截断之后的值等于128-256=-128。

[pointer]: ../types/pointer.md
[immutable place expression context]: ../expressions.md#mutability
[coercions]: ../type-coercions.md
[drops]: ../destructors.md
[either copies or moves]: ../expressions.md#moved-and-copied-types
<!-- 上面这几个链接从原文来替换时小心 -->
[place expression]: ../expressions.md#place-expressions-and-value-expressions
[value expression]: ../expressions.md#place-expressions-and-value-expressions
[temporary value]: ../expressions.md#temporaries
[float-float]: https://github.com/rust-lang/rust/issues/15536
[`unit` type]: ../types/tuple.md
[Function pointer]: ../types/function-pointer.md
[Function item]: ../types/function-item.md

[_BorrowExpression_]: #borrow-operators
[_DereferenceExpression_]: #the-dereference-operator
[_ErrorPropagationExpression_]: #the-question-mark-operator
[_NegationExpression_]: #negation-operators
[_ArithmeticOrLogicalExpression_]: #arithmetic-and-logical-binary-operators
[_ComparisonExpression_]: #comparison-operators
[_LazyBooleanExpression_]: #lazy-boolean-operators
[_TypeCastExpression_]: #type-cast-expressions
[_AssignmentExpression_]: #assignment-expressions
[_CompoundAssignmentExpression_]: #compound-assignment-expressions

[_Expression_]: ../expressions.md
[_TypeNoBounds_]: ../types.md#type-expressions
<!-- 2020-11-3 -->
<!-- checked -->
