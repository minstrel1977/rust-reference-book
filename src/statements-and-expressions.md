# 语句和表达式

>[statements-and-expressions.md](https://github.com/rust-lang/reference/blob/master/src/statements-and-expressions.md)\
>commit: 4a2bdf896cd2df370a91d14cb8ba04e326cd21db

Rust *主要*是一种表达式语言。这意味着大多数形式的产生值或产生表达效果的计算的都是由有统一的句法分类的*表达式*来控制的。每一种表达式通常都可以*内嵌*到另一种表达式中，表达式的求值规则包括指定表达式产生的值和子表达式本身求值的顺序。

对比之下，Rust 中的语句则*主要*用于包含并显式地对有序表达式进行求值。
