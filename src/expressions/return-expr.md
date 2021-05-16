# 返回(`return`)表达式

>[return-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/return-expr.md)\
>commit: eb5290329316e96c48c032075f7dbfa56990702b \
>本章译文最后维护日期：2021-02-21

> **<sup>句法</sup>**\
> _ReturnExpression_ :\
> &nbsp;&nbsp; `return` [_Expression_]<sup>?</sup>

返回(return)表达式使用关键字 `return` 来标识。
对返回(`return`)表达式求值会将其参数移动到当前函数调用的指定输出位置，然后销毁当前函数的激活帧(activation frame)，并将控制权转移到此函数的调用帧(caller frame)。

一个返回(`return`)表达式的例子：

```rust
fn max(a: i32, b: i32) -> i32 {
    if a > b {
        return a;
    }
    return b;
}
```

[_Expression_]: ../expressions.md

<!-- 2020-11-12-->
<!-- checked -->
