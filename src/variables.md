# 变量

>[variables.md](https://github.com/rust-lang/reference/blob/master/src/variables.md)\
>commit: 79fcc6e4453919977b8b3bdf5aee71146c89217d \
>本章译文最后维护日期：2020-11-16

*变量*是栈帧里的一个组件，可以是具名函数参数、匿名的[临时变量](expressions.md#temporaries)或具名局部变量。

*局部变量*（或*本地栈(stack-local)*分配）直接持有一个值，该值在栈内存中分配。该值是栈帧的一部分。

局部变量是不可变的，除非特别声明。例如：`let mut x = ...`。

函数参数是不可变的，除非用 `mut` 声明。关键字 `mut` 只应用于紧跟着它的那个参数。例如：`|mut x, y|` 和 `fn f(mut x: Box<i32>, y: Box<i32>)` 声明了一个可变变量 `x` 和一个不可变变量 `y`。

分配时不会初始化局部变量。此处一反常态的是在帧建立时，以未初始化状态分配整个帧值的局部变量。函数中的后续语句可以初始化局部变量，也可以不初始化局部变量。局部变量只有在通过所有可到达的控制流路径初始化后才能使用

在下面示例中，`init_after_if` 是在 [`if`表达式][`if` expression]执行后被初始化的，而 `uninit_after_if` 不是，因为它没有在 `else` 分支里被初始化。

```rust
# fn random_bool() -> bool { true }
fn initialization_example() {
    let init_after_if: ();
    let uninit_after_if: ();

    if random_bool() {
        init_after_if = ();
        uninit_after_if = ();
    } else {
        init_after_if = ();
    }

    init_after_if; // ok
    // uninit_after_if; // 错误：使用可能未初始化的 `uninit_after_if`
}
```

[`if` expression]: expressions/if-expr.md#if-expressions

<!-- 2020-11-12-->
<!-- checked -->
