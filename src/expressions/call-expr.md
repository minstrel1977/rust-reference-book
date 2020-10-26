# Call expressions
# 调用表达式

>[call-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/call-expr.md)\
>commit: e06136b1cd06d3d72dcc0ed7ccf5fbab5574f901 \
>本译文最后维护日期：2020-10-26

> **<sup>句法</sup>**\
> _CallExpression_ :\
> &nbsp;&nbsp; [_Expression_] `(` _CallParams_<sup>?</sup> `)`
>
> _CallParams_ :\
> &nbsp;&nbsp; [_Expression_]&nbsp;( `,` [_Expression_] )<sup>\*</sup> `,`<sup>?</sup>

*调用表达式*由一个表达式和一个圆括号封闭的表达式列表组成。它调用一个函数，并给其提供零个或多个输入变量。如果函数最终返回，则该表达式执行完成。对于[非函数类型](../types/function-item.md)，表达式 `f(...)` 会使用 [`std::ops::Fn`]、[`std::ops::FnMut`] 或 [`std::ops::FnOnce`] 这些 trait 上的方法，选择使用其中的哪些 trait 要看 `f` 如何获取其输入参数的，具体就是看是通过引用、可变引用、还是通过获取所有权来获取的。如有需要，也可通过自动借用。Rust 也会根据需要自动对 `f` 作解引用处理。下面是一些调用表达式的示例：

```rust
# fn add(x: i32, y: i32) -> i32 { 0 }
let three: i32 = add(1i32, 2i32);
let name: &'static str = (|| "Rust")();
```

## Disambiguating Function Calls
## 消除函数调用歧义

为获得更直观的、[完全限定的句法][fully-qualified syntax]，Rust 对所有函数调都作了糖化表达处理，以方便代码编写。而在编译时，Rust 又会把所有函数调用进行脱糖(desugar)处理，转换为显式形式。Rust 有时可能要求程序员使用 trait 来限定具体函数调用，这取决于调用作用域内的数据项时是否出现了模糊性。

> **注意**：过去，Rust 社区在文档、议题、RFC 和其他社区文章中使用了术语“确定性函数调用句法(Unambiguous Function Call Syntax)”、“通用函数调用句法(Universal Function Call Syntax)” 或 “UFCS”。但是，这个术语缺乏描述力，可能会混淆当前的议题。我们在这里提起它是为了便于搜索。

少数几种情况下经常会出现一些导致方法或关联函数调用的接收者或引用对象不明确的情况。这些情况可包括：

* 作用域内多个 trait 为同一类型定义了相同名称的方法
* 自动解引(Auto-`deref`)用搞不定的；例如，区分智能指针本身的方法和指针所指对象上的方法
* 不带接受者的方法，如 [`default()`]；返回类型的属性(properties)，如 [`size_of()`]

为了解决这种模糊性，程序员可以使用更具体的路径、类型或 trait 来明确指代他们想要的方法或函数。

例如：

```rust
trait Pretty {
    fn print(&self);
}

trait Ugly {
  fn print(&self);
}

struct Foo;
impl Pretty for Foo {
    fn print(&self) {}
}

struct Bar;
impl Pretty for Bar {
    fn print(&self) {}
}
impl Ugly for Bar{
    fn print(&self) {}
}

fn main() {
    let f = Foo;
    let b = Bar;

    // 我们可以这样做，因为对于`Foo`，我们只有一个名为 `print` 的数据项
    f.print();
    // 对于 `Foo`来说，是更明确了，但没必要
    Foo::print(&f);
    // 如果你不喜欢简洁的话，那，也可以这样
    <Foo as Pretty>::print(&f);

    // b.print(); // 错误： 发现多个 `print`
    // Bar::print(&b); // 仍错： 发现多个 `print`

    // 必要，因为作用域内的多个数据项定义了 `print`
    <Bar as Pretty>::print(&b);
}
```

更多细节和动机说明请参考[RFC 132]。

[RFC 132]: https://github.com/rust-lang/rfcs/blob/master/text/0132-ufcs.md
[_Expression_]: ../expressions.md
[`default()`]: https://doc.rust-lang.org/std/default/trait.Default.html#tymethod.default
[`size_of()`]: https://doc.rust-lang.org/std/mem/fn.size_of.html
[`std::ops::FnMut`]: https://doc.rust-lang.org/std/ops/trait.FnMut.html
[`std::ops::FnOnce`]: https://doc.rust-lang.org/std/ops/trait.FnOnce.html
[`std::ops::Fn`]: https://doc.rust-lang.org/std/ops/trait.Fn.html
[fully-qualified syntax]: ../paths.md#qualified-paths

<!-- 2020-10-25 -->
<!-- checked -->
