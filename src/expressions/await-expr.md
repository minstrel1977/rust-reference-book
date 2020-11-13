# Await expressions
# 等待(await)表达式

>[await-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/await-expr.md)\
>commit: f8e76ee9368f498f7f044c719de68c7d95da9972 \
>本章译文最后维护日期：2020-11-13

> **<sup>句法</sup>**\
> _AwaitExpression_ :\
> &nbsp;&nbsp; [_Expression_] `.` `await`

等待(await)表达式仅在[异步上下文][async context]中才能使用，例如 [异步函数(`async fn`)][`async fn`] 或 [异步(`async`)块][`async` block]。等待(await)表达式内部操作一个 [future] 。它的作用是挂起当前计算，直到给定的 future 准备好生成返回值。

更具体地说 `<expr>.await`表达式有以下效果。

1. 计算 `<expr>` 到一个 [future] `tmp` 中；
2. 使用 [`Pin::new_unchecked`] 固定住(Pin)这个 `tmp`；
3. 然后通过调用 [`Future::poll`] 方法对这个固定住的 future 进行轮询，同事将当前[任务上下文](#task-context)传递给它；
4. 如果轮询(`poll`)调用返回 [`Poll::Pending`]，那么这个 future 就也返回 `Poll::Pending`，并相应地挂起它的状态，这样，当包围它的异步上下文被再次轮询时，执行流返回到步骤2；
5. 否则，轮询(`poll`)调用必定返回了 [`Poll::Ready`]，在这种情况下，[`Poll::Ready`]变体中包含的值将被当作这个 `await`表达式本身的求值结果。

[`async fn`]: ../items/functions.md#async-functions
[`async` block]: block-expr.md#async-blocks
[future]: https://doc.rust-lang.org/std/future/trait.Future.html
[_Expression_]: ../expressions.md
[`Future::poll`]: https://doc.rust-lang.org/std/future/trait.Future.html#tymethod.poll
[`Context`]: https://doc.rust-lang.org/std/task/struct.Context.html
[`Pin::new_unchecked`]: https://doc.rust-lang.org/std/pin/struct.Pin.html#method.new_unchecked
[`Poll::Pending`]: https://doc.rust-lang.org/std/task/enum.Poll.html#variant.Pending
[`Poll::Ready`]: https://doc.rust-lang.org/std/task/enum.Poll.html#variant.Ready

> **版本差异**： 等待(await)表达式只能从 Rust 2018 版开始才可用

## Task context
## 任务上下文

任务上下文是指在对[异步上下文][async context]本身进行轮询时提供给当前异步上下文的[上下文(`Context`)][`Context`]。因为等待(`await`)表达式只能在异步上下文中才能使用，所以此时必须有一些任务上下文可用。

[`Context`]: https://doc.rust-lang.org/std/task/struct.Context.html
[async context]: ../expressions/block-expr.md#async-context

## Approximate desugaring
## 近似脱糖

实际上,一个 `<expr>.await`表达式大致相当于如下代码（此脱糖不规范）：

<!-- ignore: example expansion -->
```rust,ignore
match /* <expr> */ {
    mut pinned => loop {
        let mut pin = unsafe { Pin::new_unchecked(&mut pinned) };
        match Pin::future::poll(Pin::borrow(&mut pin), &mut current_context) {
            Poll::Ready(r) => break r,
            Poll::Pending => yield Poll::Pending,
        }
    }
}
```

其中，`yield`伪代码返回 `Poll::Pending`，当再次调用时，从该点继续执行。变量 `current_context` 是指从异步环境中获取的上下文。

<!-- 2020-11-12-->
<!-- checked -->
