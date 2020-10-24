# Block expressions
# 块表达式

>[block-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/block-expr.md)\
>commit: 6b90080371ff44d0074a465945dfdb0de4b50774 \
>本译文最后维护日期：2020-10-24

> **<sup>句法</sup>**\
> _BlockExpression_ :\
> &nbsp;&nbsp; `{`\
> &nbsp;&nbsp; &nbsp;&nbsp; [_InnerAttribute_]<sup>\*</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp; _Statements_<sup>?</sup>\
> &nbsp;&nbsp; `}`
>
> _Statements_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_Statement_]<sup>\+</sup>\
> &nbsp;&nbsp; | [_Statement_]<sup>\+</sup> [_ExpressionWithoutBlock_]\
> &nbsp;&nbsp; | [_ExpressionWithoutBlock_]

*块表达式*或*块*是数据项和变量声明的控制流表达式和匿名命名空间作用域。作为控制流表达式，块按顺序执行其非数据项声明的语句组件，最后执行可选的尾部表达式(final expression)。作为一个匿名命名空间作用域，在本块内声明的数据项只在块本身围成的作用域内有效，而由 `let` 语句声明的变量的作用域为下一条语句到块末尾。

块的书写形式为：先是一个 `{`，然后是[内部属性][inner attributes]，再后是[语句][statements]，再后是一个可选表达式，最后是一个 `}`。语句通常需要后跟分号，但有两个例外。数据项声明语句不需要后跟分号。表达式语句通常需要后面的分号，但它的外层表达式是流控制表达式时不需要。此外，允许在语句之间使用额外的分号，但是这些分号并不影响语义。

在对块表达式求值时，除了数据项声明语句外，每个语句都是按顺序执行的。如果给出了块尾的可选的最终表达式(final expression)，则最后会执行它。

块的类型是最终表达式的类型，但如果省略了最终表达式(final expression)，则块的类型为 `()`。
As a control flow
expression, a block sequentially executes its component non-item declaration
statements and then its final optional expression. As an anonymous namespace
scope, item declarations are only in scope inside the block itself and variables
declared by `let` statements are in scope from the next statement until the end
of the block.

Blocks are written as `{`, then any [inner attributes], then [statements],
then an optional expression, and finally a `}`. Statements are usually required
to be followed by a semicolon, with two exceptions. Item declaration statements do
not need to be followed by a semicolon. Expression statements usually require
a following semicolon except if its outer expression is a flow control
expression. Furthermore, extra semicolons between statements are allowed, but
these semicolons do not affect semantics.

When evaluating a block expression, each statement, except for item declaration
statements, is executed sequentially. Then the final expression is executed,
if given.

The type of a block is the type of the final expression, or `()` if the final
expression is omitted.

```rust
# fn fn_call() {}
let _: () = {
    fn_call();
};

let five: i32 = {
    fn_call();
    5
};

assert_eq!(5, five);
```

> 注意：作为控制流表达式，如果块表达式是表达式语句的外层表达式，则该块表达式的预期类型为 `()` ，除非它后面紧跟着一个分号。

块总是[值表达式][value expressions]，并会在值表达式上下文中对最终表达式求值。如果确实有需要，块可以用于强制移动值。例如，下面的示例在调用 `consume_self` 时失败，因为结构体已经在之前的块表达式里被从 `s` 里移出了。

```rust,compile_fail
struct Struct;

impl Struct {
    fn consume_self(self) {}
    fn borrow_self(&self) {}
}

fn move_by_block_expression() {
    let s = Struct;

    // 将值从块表达式里的 `s` 里移出。
    (&{ s }).borrow_self();

    // 执行失败，因为 `s` 里的值已经被移出。
    s.consume_self();
}
```

## `async`块

> **<sup>句法</sup>**\
> _AsyncBlockExpression_ :\
> &nbsp;&nbsp; `async` `move`<sup>?</sup> _BlockExpression_

*异步块(async block)*是计算为 *future* 的块表达式。块的最终表达式(如果存在)决定了 future 的结果值。（译者注：单词future对应中文为“未来”。原文里，作为类型的future和字面意义上的future经常混用，所以译者基本保留此单词不翻译，特别强调“未来”的意义时也会加上其英文单词）

执行一个异步块类似于执行一个闭包表达式：它的直接效果是生成并返回一个匿名类型。类似闭包返回的类型实现了一个或多个 [`std::ops::Fn`] trait，异步块返回的类型实现了 [`std::future::Future`] trait。此类型的实际数据格式的规范还未确定下来。

> **注意：** rustc 生成的 future类型大致相当于为每个 `await` 点生成一个枚举变体，其中每个变体都存储了从对应点捕获的数据。

> **版本差异**: 异步块从 Rust 2018版开始可用。

[`std::ops::Fn`]: https://doc.rust-lang.org/std/ops/trait.Fn.html
[`std::future::Future`]: https://doc.rust-lang.org/std/future/trait.Future.html

### 捕获模式

异步块使用与闭包相同的[捕获模式]从其环境中捕获变量。就像闭包，当编写 `async { .. }` 时，每个变量的捕获模式将从该块里的内容中推断出来。
而 `async move { .. }` 类型的异步块将把所有引用的变量移入(move)到相应的 future 中。

[捕获模式]: ../types/closure.md#capture-modes

### 异步上下文

因为异步块构造了一个 future，所以它们定义了一个**async上下文**，这个上下文可以相应地包含 [`await`表达式]。异步上下文是由异步块和异步函数体建立的，它们的语义是根据异步块定义的。
<!-- Because async blocks construct a future, they define an **async context** which can in turn contain [`await` expressions].  Async contexts are established by async blocks as well as the bodies of async functions, whose semantics are defined in terms of async blocks. TobeModify-->

[`await` expressions]: await-expr.md

### 控制流操作符

异步块和函数边界类似，或者更类似于闭包。因此 `?`操作符符和 `return`表达式能影响 future 的输出，而不会影响封闭它的函数或其他上下文。也就是说，future 的输出跟闭包将其中的 `return <expr>` 的表达式 `<expr>` 的计算结果作为输出的做法是一样的。类似地,如果 `<expr>?` 传播一个错误，这个错误会作为未来(future)的结果传播出去。

最后，`break` 和 `continue` 关键字不能用于从异步块中跳出分支。因此，以下内容是非法的：

```rust,edition2018,compile_fail
loop {
    async move {
        break; // 这将打破循环。
    }
}
```

##非安全(`unsafe`)块

> **<sup>句法</sup>**\
> _UnsafeBlockExpression_ :\
> &nbsp;&nbsp; `unsafe` _BlockExpression_

_查看 [`unsafe`块](../unsafe-blocks.md)以了解更多该何时使用` unsafe` 的信息_

可以在代码块前面加上 `unsafe` 关键字以允许[非安全操作][unsafe operations]。例如：

```rust
unsafe {
    let b = [13u8, 17u8];
    let a = &b[0] as *const u8;
    assert_eq!(*a, 13);
    assert_eq!(*a.offset(1), 17);
}

# unsafe fn an_unsafe_fn() -> i32 { 10 }
let a = unsafe { an_unsafe_fn() };
```

## 块表达式上的属性

在以下情况下，允许在块表达式的左括号之后直接使用[内部属性][inner attributes]：

* [函数][function]和[方法][method]的代码体。
* 循环体([`loop`], [`while`], [`while let`], 和 [`for`])。
* 块表达式用作[语句]。
* 块表达式作为[数组表达式][array expressions]、[元组表达式][tuple expressions]、[调用表达式][call expressions]、[元组结构体][struct]和[枚举变体][enum variant]表达式的元素。
* 作为另一个块表达式的尾表达式的块表达式。
<!-- 本列表需要和 expressions.md 保持同步 -->

The attributes that have meaning on a block expression are [`cfg`] and [the lint check attributes].

For example, this function returns `true` on unix platforms and `false` on other platforms.

```rust
fn is_unix_platform() -> bool {
    #[cfg(unix)] { true }
    #[cfg(not(unix))] { false }
}
```

[_ExpressionWithoutBlock_]: ../expressions.md
[_InnerAttribute_]: ../attributes.md
[_Statement_]: ../statements.md
[`cfg`]: ../conditional-compilation.md
[`for`]: loop-expr.md#iterator-loops
[`loop`]: loop-expr.md#infinite-loops
[`while let`]: loop-expr.md#predicate-pattern-loops
[`while`]: loop-expr.md#predicate-loops
[array expressions]: array-expr.md
[call expressions]: call-expr.md
[enum variant]: enum-variant-expr.md
[function]: ../items/functions.md
[inner attributes]: ../attributes.md
[method]: ../items/associated-items.md#methods
[statement]: ../statements.md
[statements]: ../statements.md
[struct]: struct-expr.md
[the lint check attributes]: ../attributes/diagnostics.md#lint-check-attributes
[tuple expressions]: tuple-expr.md
[unsafe operations]: ../unsafety.md
[value expressions]: ../expressions.md#place-expressions-and-value-expressions

<!-- 2020-10-16 -->
<!-- checked -->
