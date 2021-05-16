# 等待(await)表达式

>[await-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/await-expr.md)\
>commit: 23672971a16c69ea894bef24992b74912cfe5d25 \
>本章译文最后维护日期：2021-4-5

> **<sup>句法</sup>**\
> _AwaitExpression_ :\
> &nbsp;&nbsp; [_Expression_] `.` `await`

*等待(await)表达式*挂起当前计算，直到给定的 future 准备好生成值。
等待(await)表达式的句法格式为：一个其类型实现了 [Future] trait 的表达式（此表达式本身被称为 *future操作数*）后跟一 `.`标记，再后跟一个 `await`关键字。

等待(await)表达式仅在[异步上下文][async context]中才能使用，例如 [异步函数(`async fn`)][`async fn`] 或 [异步(`async`)块][`async` block]。

更具体地说，等待(await)表达式具有以下效果：

1. 把 future操作数求值计算到一个 [future]类型的 `tmp` 中；
2. 使用 [`Pin::new_unchecked`] 固定住(Pin)这个 `tmp`；
3. 然后通过调用 [`Future::poll`] 方法对这个固定住的 future 进行轮询，同事将当前[任务上下文](#task-context)传递给它；
4. 如果轮询(`poll`)调用返回 [`Poll::Pending`]，那么这个 future 就也返回 `_Expression_

> **版本差异**： 等待(await)表达式只能从 Rust 2018 版开始才可用

## 任务上下文

任务上下文是指在对[异步上下文][async context]本身进行轮询时提供给当前异步上下文的[上下文(`Context`)][`Context`]。
因为等待(`await`)表达式只能在异步上下文中才能使用，所以此时必须有一些任务上下文可用。

## 近似脱糖

实际上,一个等待(await)表达式大致相当于如下这个非正规的脱糖过程：

<!-- ignore: example expansion -->
```rust,ignore
match future_operand {
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
[`context`]: https://doc.rust-lang.org/std/task/struct.Context.html
[`future::poll`]: https://doc.rust-lang.org/std/future/trait.Future.html#tymethod.poll
[`pin::new_unchecked`]: https://doc.rust-lang.org/std/pin/struct.Pin.html#method.new_unchecked
[`poll::Pending`]: https://doc.rust-lang.org/std/task/enum.Poll.html#variant.Pending
[`poll::Ready`]: https://doc.rust-lang.org/td/task/enum.Poll.html#variant.Ready
[async context]: ../expressions/block-expr.md#async-context
[future]: https://doc.rust-lang.org/std/future/trait.Future.html
