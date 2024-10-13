# Input format
# 输入格式

>[notation.md](https://github.com/rust-lang/reference/blob/master/src/input-format.md)\
>commit: fa5f313ac63c910ca9a6158fc7a4995a30b452e5 \
>本章译文最后维护日期：2024-10-13

r[input]

r[input.intro]
本章介绍如何将源文件解释为一系列 token。

有关程序如何组织成文件的描述，请参见[crate 和源文件][Crates and source files]。

## Source encoding
## 源文件编码方式

r[input.encoding]

r[input.encoding.utf8]
每个源文件都被解释为以 UTF-8编码的 Unicode字符序列。

r[input.encoding.invalid]
如果文件不是有效的 UTF-8编码方式，则报错。

## Byte order mark removal
## BOM移除

r[input.byte-order-mark]
如果字符序列中的第一个字符是 `U+FEFF` ([BYTE ORDER MARK])，则会将其移除。

## CRLF normalization
## CRLF归一化

r[input.crlf]

`U+000D`(CR) 后紧跟着 `U+000A`(LF) 这样的字符对都被一个 `U+000A`(LF) 所代替。

其他时候出现的字符`U+000D`(CR) 则保留在原位（它们被视为[空白符][whitespace]）。

## Shebang removal
## Shebang 移除

r[input.shebang]

r[input.shebang.intro]
如果字符序列以字符 `#!` 起行，那直到并包括第一个 `U+000A`(LF) 的字符将被从此字符序列中移除。

例如，以下文件的第一行将被忽略：

<!-- ignore: tests don't like shebang -->
```rust,ignore
#!/usr/bin/env rustx

fn main() {
    println!("Hello!");
}
```

r[input.shebang.inner-attribute]
作为例外，如果 `#!` 字符后面跟着一个 `[`token（此时需要忽略中间的[注释][comments]或[空白符][whitespace]），则不会删除任何内容。

这样可以防止删除了源文件开头的[内部属性][inner attribute]。

> **注意**： 标准库的 [`include!`]宏读取的文件会采用BOM移除、CRLF归一化和 shebang删除的处理方式。但 [`include_str!`] 和[`include_bytes!`]宏没有采用。

## Tokenization
## token化

r[input.tokenization]

源文件被前面步骤处理后的的字符序列将如本章剩余部分所述那样被转换为 token。

[inner attribute]: attributes.md
[BYTE ORDER MARK]: https://en.wikipedia.org/wiki/Byte_order_mark#UTF-8
[comments]: comments.md
[Crates and source files]: crates-and-source-files.md
[_shebang_]: https://en.wikipedia.org/wiki/Shebang_(Unix)
[whitespace]: whitespace.md
