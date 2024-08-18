# Block expressions
# 块表达式

>[block-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/block-expr.md)\
>commit: 04a0845417fddbbd5cbd453241d51798e70442b3 \
>本章译文最后维护日期：2024-08-17

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

*块表达式*或*块*是一个控制流表达式(control flow expression)，同时也是程序项声明和变量声明的匿名空间作用域。
作为控制流表达式，块按顺序执行其非程序项声明的语句组件，最后执行可选的最终表达式(final expression)。
作为一个匿名空间作用域，在本块内声明的程序项只在块本身围成的作用域内有效，而块内由 `let`语句声明的变量的作用域为下一条语句到块尾。
更多细节请参见[作用域][scopes]章节。

块的句法规则为：先是一个 `{`，后跟[内部属性][inner attributes]，再后是任意条[语句][statements]，再后是一个被称为最终操作数（final operand）的可选表达式，最后是一个 `}`。

语句之间通常需要后跟分号，但有两个例外：
1、程序项声明语句不需要后跟分号。
2、表达式语句通常需要后面的分号，但它的外层表达式是控制流表达式时不需要。

此外，允许在语句之间使用额外的分号，但是这些分号并不影响语义。

在对块表达式进行求值时，除了程序项声明语句外，每个语句都是按顺序执行的。
如果给出了块尾的可选的最终操作数(final operand)，则最后会执行它。

块的类型是最此块的最终操作数(final operand)的类型，但如果省略了最终操作数，则块的类型为 `()`。

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

> 注意：作为控制流表达式，如果块表达式是一个表达式语句的外层表达式，则该块表达式的预期类型为 `()`，除非该块后面紧跟着一个分号。

块总是[值表达式][value expressions]，并会在值表达式上下文中对最后的那个操作数进行求值。

> **注意**：如果确实有需要，块可以用于强制移动值。
> 例如，下面的示例在调用 `consume_self` 时失败，因为结构体已经在之前的块表达式里被从 `s` 里移出了。
> 
> ```rust,compile_fail
> struct Struct;
> 
> impl Struct {
>     fn consume_self(self) {}
>     fn borrow_self(&self) {}
> }
> 
> fn move_by_block_expression() {
>     let s = Struct;
> 
>     // 将值从块表达式里的 `s` 里移出。
>     (&{ s }).borrow_self();
> 
>     // 执行失败，因为 `s` 里的值已经被移出。
>     s.consume_self();
> }
> ```

## `async` blocks
## `async`块

> **<sup>句法</sup>**\
> _AsyncBlockExpression_ :\
> &nbsp;&nbsp; `async` `move`<sup>?</sup> _BlockExpression_

*异步块(async block)*是求值为 *future* 的块表达式的一个变体。
块的最终表达式（如果存在）决定了 future 的结果值。（译者注：单词 future 对应中文为“未来”。原文可能为了双关的行文效果，经常把作为类型的 future 和字面意义上的 future 经常混用，所以译者基本保留此单词不翻译，特别强调“未来”的意义时也会加上其英文单词。）

执行一个异步块类似于执行一个闭包表达式：它的即时效果是生成并返回一个匿名类型。
类似闭包返回的类型实现了一个或多个 [`std::ops::Fn`] trait，异步块返回的类型实现了 [`std::future::Future`] trait。
此类型的实际数据格式规范还未确定下来。

> **注意：** rustc 生成的 future类型大致相当于一个枚举，rustc 为这个 future 的每个 `await`点生成一个此枚举的变体，其中每个变体都存储了对应点再次恢复执行时需要的数据。

> **版次差异**: 异步块从 Rust 2018 版才开始可用。

### Capture modes
### 捕获方式

异步块使用与闭包相同的[捕获方式][capture modes]从其环境中捕获变量。
跟闭包一样，当编写 `async { .. }` 时，每个变量的捕获方式将从该块里的内容中推断出来。
而 `async move { .. }` 类型的异步块将把所有需要捕获的变量使用移动语义移入(move)到相应的结果 future 中。

### Async context
### 异步上下文

因为异步块构造了一个 future，所以它们定义了一个**async上下文**，这个上下文可以相应地包含 [`await`表达式][`await` expressions]。
异步上下文是由异步块和异步函数的函数体建立的，它们的语义是依照异步块定义的。


### Control-flow operators
### 控制流操作符

异步块的作用类似于函数的边界符来界定函数，或者更类似于闭包。
因此 `?`操作符和 返回(`return`)表达式也都能影响 future 的输出，且都不会影响封闭它的函数或其他上下文。
也就是说，`return <expr>` 在异步块中跟其在 future 的输出是一样的，都是将 `<expr>` 的计算结果返回。
类似地,如果 `<expr>?` 传播(propagate)一个错误，这个错误也会被 future 在未来的某个时候作为返回结果被传播出去。

最后，关键字 `break` 和 `continue` 不能用于从异步块中跳出分支。
因此，以下内容是非法的：

```rust,compile_fail
loop {
    async move {
        break; // 错误[E0267]: `break` inside of an `async` block
    }
}
```


## `const` blocks
## Const块

> **<sup>句法</sup>**\
> _ConstBlockExpression_ :\
> &nbsp;&nbsp; `const` _BlockExpression_

*Const块*是块表达式的变体，其代码主体在编译时而不是在运行时求值。

Const块允许你直接一个定义常量值，而不必定义新的[常量项][constant items]，因此它们有时也称为*内连常量*。
Const块还支持类型推理，因此不需要指定类型，这点与[常量项][constant items]不同。

与[自由][free item]常量项不同，Const块能够引用作用域中的泛型参数。
它们会被脱糖为作用域中带有泛型参数的常量项（类似于关联常量，但没有与它们关联的 trait 或类型）。
例如，下面这些代码：

```rust
fn foo<T>() -> usize {
    const { std::mem::size_of::<T>() + 1 }
}
```

等价于：

```rust
fn foo<T>() -> usize {
    {
        struct Const<T>(T);
        impl<T> Const<T> {
            const CONST: usize = std::mem::size_of::<T>() + 1;
        }
        Const::<T>::CONST
    }
}
```

如果在运行时执行 Const块表达式，则会确保其常量已经被求值，即使其返回值会被忽略：
If the const block expression is executed at runtime, then the constant is guaranteed to be evaluated, even if its return value is ignored:

```rust
fn foo<T>() -> usize {
    // 如果这段代码被执行，那么断言的值肯定是在编译时就被求值了。
    const { assert!(std::mem::size_of::<T>() > 0); }
    // Here we can have unsafe code relying on the type being non-zero-sized.
    // 这里，我们就可以放置一些依赖于非零尺寸类型的 unsafe代码。
    /* ... */
    42
}
```

如果在运行时不会执行 Const块表达式，则编译时可以计算它，也可以不计算它：
```rust,compile_fail
if false {
    // 在构建程序时，此panic 有可能触发，也有可能不触发。
    const { panic!(); }
}
```

## `unsafe` blocks
## 非安全(`unsafe`)块

> **<sup>句法</sup>**
> _UnsafeBlockExpression_ :\
> &nbsp;&nbsp; `unsafe` _BlockExpression_

_参见 [`unsafe`块][`unsafe` blocks] 以查看更多和 `unsafe` 相关的信息_

可以在代码块前面加上关键字 `unsafe` 以允许[非安全操作][unsafe operations]。
例如：

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

## Labelled block expressions
## 带标签的块表达式

带标签的块表达式的相关文档记录在[循环和其他可中断表达式][Loops and other breakable expressions]的相关章节中。
## Attributes on block expressions

在以下上下文中，允许在块表达式的左括号之后直接使用[内部属性][inner attributes]：

* [函数][function]和[方法][method]的代码体。
* 循环体（[`loop`], [`while`], [`while let`], 和 [`for`]）。
* 被用作[语句][statement]的块表达式。
* 块表达式作为[数组表达式][array expressions]、[元组表达式][tuple expressions]、[调用表达式][call expressions]、[元组结构体][struct]表达式和[枚举变体][enum variant]表达式的元素。
* 作为另一个块表达式的尾部表达式(tail expression)的块表达式。
<!-- 本列表需要和 expressions.md 保持同步 -->

在块表达式上有意义的属性有 [`cfg`] 和 [lint检查类属性][the lint check attributes]。

例如，下面这个函数在 unix 平台上返回 `true`，在其他平台上返回 `false`。

```rust
fn is_unix_platform() -> bool {
    #[cfg(unix)] { true }
    #[cfg(not(unix))] { false }
}
```

[_ExpressionWithoutBlock_]: ../expressions.md
[_InnerAttribute_]: ../attributes.md
[_Statement_]: ../statements.md
[`await` expressions]: await-expr.md
[`cfg`]: ../conditional-compilation.md
[`for`]: loop-expr.md#iterator-loops
[`loop`]: loop-expr.md#infinite-loops
[`std::ops::Fn`]: https://doc.rust-lang.org/std/ops/trait.Fn.html
[`std::future::Future`]: https://doc.rust-lang.org/std/future/trait.Future.html
[`unsafe` blocks]: ../unsafe-keyword.md#unsafe-blocks-unsafe-
[`while let`]: loop-expr.md#predicate-pattern-loops
[`while`]: loop-expr.md#predicate-loops
[array expressions]: array-expr.md
[call expressions]: call-expr.md
[capture modes]: ../types/closure.md#capture-modes
[constant items]: ../items/constant-items.md
[free item]: ../glossary.md#free-item
[function]: ../items/functions.md
[inner attributes]: ../attributes.md
[method]: ../items/associated-items.md#methods
[mutable reference]: ../types/pointer.md#mutables-references-
[scopes]: ../names/scopes.md
[shared references]: ../types/pointer.md#shared-references-
[statement]: ../statements.md
[statements]: ../statements.md
[struct]: struct-expr.md
[the lint check attributes]: ../attributes/diagnostics.md#lint-check-attributes
[tuple expressions]: tuple-expr.md
[unsafe operations]: ../unsafety.md
[value expressions]: ../expressions.md#place-expressions-and-value-expressions
[Loops and other breakable expressions]: loop-expr.md#labelled-block-expressions