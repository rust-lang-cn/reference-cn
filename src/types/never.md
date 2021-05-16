# never 类型

>[never.md](https://github.com/rust-lang/reference/blob/master/src/types/never.md)\
>commit: 91486df597a9e8060f9c67587359e9f168dea7ef \
>本章译文最后维护日期：2020-11-14

> **<sup>句法</sup>**\
> _NeverType_ : `!`

never类型(`!`)是一个没有值的类型，表示永远不会完成计算的结果。`!` 的类型表达式可以强转为任何其他类型。

<!-- ignore: unstable -->
```rust,ignore
let x: ! = panic!();
// 可以强转为任何类型
let y: u32 = x;
```

**注意：** never类型原本预计在1.41中稳定下来，但由于最后一分钟检测到一些意想不到的回归，该类型的稳定进程临时暂停。目前 `!`类型只能出现在函数返回类型中。有关详细信息，请参阅[议题跟踪](https://github.com/rust-lang/rust/issues/35121)。

<!-- 2020-11-12-->
<!-- checked -->
