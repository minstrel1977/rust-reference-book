# Boolean type
# 布尔型

>[boolean.md](https://github.com/rust-lang/reference/blob/master/src/types/boolean.md)\
>commit: 0a15f29adb9988fcf4a57754c820332f5b3b214a \
>本译文最后维护日期：2020-10-29

布尔(`bool`)型是一种可以为真(`true`)或假(`false`)的数据类型。布尔型使用一个字节的内存。它用于比较和按位操作，如 `&`、`|` 和 `!`。

```rust
fn main() {
    let x = true;
    let y: bool = false; // 使用布尔类型标注

    // 在条件表达式中使用布尔值
    if x {
        println!("x is true");
    }
}
```

<!-- 2020-11-3 -->
<!-- checked -->
