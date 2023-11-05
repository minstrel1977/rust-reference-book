# Textual types
# 文本类类型

>[textual.md](https://github.com/rust-lang/reference/blob/master/src/types/textual.md)\
>commit: 192178f936f0057932e117183f774f0e2cce832c \
>本章译文最后维护日期：2023-11-05

类型 `char` 和 `str` 用于保存文本数据。

字符型(`char`)的值是 [Unicode 标量(scalar)值][Unicode scalar value]（即不是代理项(surrogate)的代码点），可以表示为 0x0000~0xD7FF 或 0xE000~0x10FFFF 范围内的 32-bit 无符号字符。创建超出此范围的字符直接触发[未定义行为(Undefined Behavior)][Undefined Behavior]。一个 `[char]` 实际上是长度为1的 UCS-4 / UTF-32 字符串。

`str`类型的值的表示方法与 `[u8]` 相同，也一个 8-bit 无符号字节类型的切片。但是，Rust 标准库对 `str` 做了额外的假定：`str` 上的方法会假定并确保其中的数据是有效的 UTF-8。调用 `str` 的方法来处理非UTF-8 缓冲区上的数据可能或早或晚地出现[未定义行为][Undefined Behavior]。

由于 `str` 是一个[动态内存宽度类型][dynamically sized type]，所以它只能通过指针类型实例化，比如 `&str`。

## Layout and bit validity
## 内存布局和位有效性

`char` 在所有平台上都保证具有与 `u32` 相同的尺寸和对齐方式。

`char` 的每个字节都保证会被初始化（换句话说，`transmute::<char, [u8; size_of::<char>()]>(...)` 总是正确的——但由于某些位模式是无效的 `char`，因此相反的操作并不总是正确的）。

[Unicode scalar value]: http://www.unicode.org/glossary/#unicode_scalar_value
[Undefined Behavior]: ../behavior-considered-undefined.md
[dynamically sized type]: ../dynamically-sized-types.md
