# Textual types
# 文本类类型

>[textual.md](https://github.com/rust-lang/reference/blob/master/src/types/textual.md)\
>commit: 9af5071f876111a09ba54a86655679de83eb464c

类型 `char` 和 `str` 保存文本数据。

字符型(`char`)的值是 [Unicode 标量(scalar)值][Unicode scalar value] (即不是代理项的代码点)，表示为 0x0000 到 0xD7FF 或 0xE000 到 0x10FFFF 范围内的32位无符号字。创建超出此范围的字符直接触发[未定义行为(Undefined Behavior)][Undefined Behavior]。一个 `[char]` 实际上是长度为1的 UCS-4 / UTF-32 字符串。

`str`类型的值的表示方法与 `[u8]` 相同，它是一个8位无符号字节的切片。然而，Rust 标准库对 `str` 做了额外的假定：处理 `str` 的方法会假定并确保其中的数据是有效的 UTF-8。调用 `str`方法来处理非UTF-8 缓冲区上的数据可能或早或晚地导致[未定义行为][Undefined Behavior]。

由于 `str` 是一个[动态尺寸类型][dynamically sized type]，它只能通过指针类型实例化，比如 `&str`。

[Unicode scalar value]: http://www.unicode.org/glossary/#unicode_scalar_value
[Undefined Behavior]: ../behavior-considered-undefined.md
[dynamically sized type]: ../dynamically-sized-types.md
