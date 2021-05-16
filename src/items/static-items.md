# 静态项

>[static-items.md](https://github.com/rust-lang/reference/blob/master/src/items/static-items.md)\
>commit: 761ad774fcb300f2b506fed7b4dbe753cda88d80 \
>本章译文最后维护日期：2021-1-17

> **<sup>句法</sup>**\
> _StaticItem_ :\
> &nbsp;&nbsp; `static` `mut`<sup>?</sup> [IDENTIFIER] `:` [_Type_]
>              ( `=` [_Expression_] )<sup>?</sup> `;`

*静态项*类似于[常量项][constant]，除了它在程序中表示一个精确的内存位置。所有对静态项的引用都指向相同的内存位置。静态项拥有 `'static` 生存期，它比 Rust 程序中的所有其他生存期都要长。静态项不会在程序结束时调用析构动作 [`drop`]。

静态初始化器是在编译时求值的[常量表达式][constant expression]。静态初始化器可以引用其他静态项。

包含非[内部可变][interior mutable]类型的非 `mut` 静态项可以放在只读内存中。

所有访问静态项的操作都是安全的，但对静态项有一些限制：

* 静态项的数据类型必须有 `Sync` trait约束，这样才可以让线程安全地访问。
* 常量项不能引用静态项。

必须为自由静态项提供初始化表达式，但在[外部块][external block]中静态项必须省略初始化表达式。

## 可变静态项

如果静态项是用关键字 `mut` 声明的，则它允许被程序修改。Rust 的目标之一是使尽可能的避免出现并发 bug，那允许修改可变静态项显然是竞态(race conditions)或其他 bug 的一个重要来源。因此，读取或写入可变静态项变量时需要引入非安全(`unsafe`)块。应注意确保对可变静态项的修改相对于运行在同一进程中的其他线程来说是安全的。

尽管有这些缺点，可变静态项仍然非常有用。它们可以与 C库一起使用，也可以在外部(`extern`)块中从 C库中来绑定它。

```rust
# fn atomic_add(_: &mut u32, _: u32) -> u32 { 2 }

static mut LEVELS: u32 = 0;

// 这违反了不共享状态的思想，而且它在内部不能防止竞争，所以这个函数是非安全的(`unsafe`)
unsafe fn bump_levels_unsafe1() -> u32 {
    let ret = LEVELS;
    LEVELS += 1;
    return ret;
}

// 这里我们假设有一个返回旧值的 atomic_add 函数，这个函数是“安全的(safe)”，
// 但是返回值可能不是调用者所期望的，所以它仍然被标记为 `unsafe`
unsafe fn bump_levels_unsafe2() -> u32 {
    return atomic_add(&mut LEVELS, 1);
}
```

除了可变静态项的类型不需要实现 `Sync` trait 之外，可变静态项与普通静态项具有相同的限制。

## 使用常量项或静态项

应该使用常量项还是应该使用静态项可能会令人困惑。一般来说，常量项应优先于静态项，除非以下情况之一成立：

* 存储大量数据
* 需要静态项的存储地址不变的特性。
* 需要内部可变性。

[constant]: constant-items.md
[`drop`]: ../destructors.md
[constant expression]: ../const_eval.md#constant-expressions
[external block]: external-blocks.md
[interior mutable]: ../interior-mutability.md
[IDENTIFIER]: ../identifiers.md
[_Type_]: ../types.md#type-expressions
[_Expression_]: ../expressions.md

<!-- 2021-1-17-->
<!-- checked -->
