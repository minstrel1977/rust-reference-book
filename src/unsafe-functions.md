# Unsafe functions
# 非安全函数

>[unsafe-functions.md](https://github.com/rust-lang/reference/blob/master/src/unsafe-functions.md)\
>commit:  4a2bdf896cd2df370a91d14cb8ba04e326cd21db \
>本译文最后维护日期：2020-11-2

非安全函数是指在所有上下文和/或所有可能的输入中都不安全的函数。这样的函数必须以关键字 `unsafe` 前缀修饰，并且只能在非安全(`unsafe`)块或其他非安全(`unsafe`)函数中调用此类函数。

<!-- 2020-11-7-->
<!-- checked -->
