# Await expressions
# 等待(await)表达式

>[await-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/await-expr.md)\
>commit: 52e0ff3c11260fb86f19e564684c86560eab4ff9 \
>本章译文最后维护日期：2024-10-13

> **<sup>句法</sup>**\
> _AwaitExpression_ :\
> &nbsp;&nbsp; [_Expression_] `.` `await`

*等待(await)表达式*挂起当前计算，直到给定的 future 准备好生成值。
等待(await)表达式的句法格式为：一个其类型实现了 [Future] trait 的表达式（此表达式本身被称为 *future操作数*）后跟一 `.`标记，再后跟一个 `await`关键字。


“await”表达式是一种语法构造，用于挂起由“std:：future:：IntoFuture”实现提供的计算，直到给定的future准备好生成值。

等待表达式的语法是一个表达式，其类型实现了〔‘IntoFuture‘〕特性，称为＊future operand＊，然后是标记‘，然后是“等待”关键字。 
*等待(await)*表达式是一种语法构造，用来挂起由 `std::future::IntoFuture` 的各类实现提供的计算，直到给定的 future 准备好生成值。
等待表达式的本身有自己的类型，此类型实现了 [`IntoFuture`] trait，表达式本身被称为 *future操作数*，后跟一 token `.`，再后跟一个 `await`关键字。

等待(await)表达式仅在[异步上下文][async context]中才能使用，例如 [异步函数(`async fn`)][`async fn`] 或 [异步(`async`)块][`async` block]。

更具体地说，等待(await)表达式具有以下效果：

1. 通过在 future操作数上调用 [`IntoFuture::into_future`] 来创建一个 future。
2. 对 future 进行求值，结果保存到一个 [future]类型的 `tmp` 中；
3. 使用 [`Pin::new_unchecked`] 固定住(Pin)这个 `tmp`；
4. 然后通过调用 [`Future::poll`] 方法对这个固定住的 future 进行 poll操作，同时将当前[任务上下文](#task-context)传递给它；
5. 如果 `poll`调用返回 [`Poll::Pending`]，那么这个 future 就也返回 `Poll::Pending`，然后以此状态挂起此表达式，当外部异步上下文再次 poll 此表达式时，过程又从步骤3开始执行。
6. 如果 `poll`调用不返回 [`Poll::Pending`]，那必定返回了 [`Poll::Ready`]，这种情况下，变体[`Poll::Ready`] 中包含的值通常就是 `await`表达式自己的返回结果。

> **版次差异**： 等待(await)表达式只能从 Rust 2018 版开始才可用
zh |
## Task context
## 任务上下文

任务上下文是指在对[异步上下文][async context]本身进行 poll 时提供给当前异步上下文的[上下文(`Context`)][`Context`]。
因为等待(`await`)表达式只能在异步上下文中才能使用，所以此时必须有一些任务上下文可用。

## Approximate desugaring
## 近似脱糖

实际上,一个等待(await)表达式大致相当于如下这个非正规的脱糖过程：

<!-- ignore: example expansion -->
```rust,ignore
match operand.into_future() {
    mut pinned => loop {
        let mut pin = unsafe { Pin::new_unchecked(&mut pinned) };
        match Pin::future::poll(Pin::borrow(&mut pin), &mut current_context) {
            Poll::Ready(r) => break r,
            Poll::Pending => yield Poll::Pending,
        }
    }
}
```

其中，`yield`伪代码返回 `Poll::Pending`，当再次调用时，从该点继续执行。
变量 `current_context` 是指从异步环境中获取的上下文。

[_Expression_]: ../expressions.md
[`async fn`]: ../items/functions.md#async-functions
[`async` block]: block-expr.md#async-blocks
[`Context`]: std::task::Context
[`future::poll`]: std::future::Future::poll
[`pin::new_unchecked`]: std::pin::Pin::new_unchecked
[`poll::Pending`]: std::task::Poll::Pending
[`poll::Ready`]: std::task::Poll::Ready
[async context]: ../expressions/block-expr.md#async-context
[future]: std::future::Future
[`IntoFuture`]: std::future::IntoFuture
[`IntoFuture::into_future`]: std::future::IntoFuture::into_future

