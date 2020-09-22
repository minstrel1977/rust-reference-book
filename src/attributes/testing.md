# 测试类属性

>[testing.md](https://github.com/rust-lang/reference/blob/master/src/attributes/testing.md)\
>commit b0e0ad6490d6517c19546b1023948986578fc378

以下[属性]用于指定执行测试的函数。在“测试”模式下编译 crate 可以构建测试函数以及执行测试的测试套件。启用测试模式还会启用 [`test`条件编译选项]。

## `test`属性

*`test`属性*将一个函数作为测试函数执行。这些函数只在测试模式下编译。测试函数必须是自由函数和单态函数，不能有参数，返回类型必须是以下类型之一：

* `()`
* `Result<(), E> where E: Error`
<!-- * `!` -->
<!-- * Result<!, E> where E: Error` -->

> 注意：允许哪些返回类型的实现是由暂未稳定的 [`Termination`] trait 决定的。

<!-- 如果前面这节需要更新(从 "不能有参数" 开始, 同时需要修改 ../crates-and-source-files.md 文件 -->

> 注意：测试模式是通过将 `--test` 参数传递给 `rustc` 或使用 `cargo test` 来启用的。

返回 `()` 的测试只要终止且不 panic 就会通过。返回 `Result<(), E>` 的测试只要它们返回 `Ok(())` 就算通过。不终止的测试既不（计为）通过也不（计为）失败。

```rust
# use std::io;
# fn setup_the_thing() -> io::Result<i32> { Ok(1) }
# fn do_the_thing(s: &i32) -> io::Result<()> { Ok(()) }
#[test]
fn test_the_thing() -> io::Result<()> {
    let state = setup_the_thing()?; // 预期成功
    do_the_thing(&state)?;          // 预期成功
    Ok(())
}
```

## `ignore`属性

用 `test` 属性注解的函数也可以用 `ignore` 属性注解。*`ignore`属性*告诉测试工具不要将该函数作为测试执行。但在测试模式下，这类函数仍然会被编译。

`ignore`属性可以选择使用[_MetaNameValueStr_]句法格式来指定测试被忽略的原因。

```rust
#[test]
#[ignore = "not yet implemented"]
fn mytest() {
    // …
}
```

> **注意**：`rustc`测试套件支持使用 `--include-ignored` 标志来强制运行被 `ignore`属性注解的测试函数。

## `should_panic`属性

用 `test` 属性注解并返回 `()` 的函数也可以用 `should_panic` 属性注解。*`should_panic`属性*使测试函数只有在实际发生 panic 时才算通过。

`should_panic`属性可选输入一条出现在 panic 返回消息中的字符串。如果在返回消息中找不到该字符串，则测试将失败。可以使用[_MetaNameValueStr_]句法格式或带有 `expected` 字段的[_MetaListNameValueStr_]句法格式来传递字符串。

```rust
#[test]
#[should_panic(expected = "values don't match")]
fn mytest() {
    assert_eq!(1, 2, "values don't match");
}
```

[_MetaListNameValueStr_]: ../attributes.md#元数据项属性句法
[_MetaNameValueStr_]: ../attributes.md#元数据项属性句法
[`Termination`]: https://doc.rust-lang.org/std/process/trait.Termination.html
[`test`条件编译选项]: ../conditional-compilation.md#test
[属性]: ../attributes.md