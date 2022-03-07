# 内部可变性

>[interior-mutability.md](https://github.com/rust-lang/reference/blob/master/src/interior-mutability.md)\
>commit: e7dd3618d78928322f53a20e2947e428b12eda2b \
>本章译文最后维护日期：2020-11-15

有时一个类型需要在存在多个别名时进行更改。在 Rust 中，这是通过一种叫做*内部可变性*的模式实现的。如果一个类型的内部状态可以通过对它的[共享引用][shared reference]来进行更改，那么就说这个类型就具有内部可变性。这违背了共享引用所指向的值不能被更改的通常[要求][ub]。

[`std::cell::UnsafeCell<T>`]类型是 Rust 中唯一可以合法实现此要求的方法。当 `UnsafeCell<T>` 存在其他不变性的别名[^译注1]时，仍然可以安全地对它包含的 `T` 进行更改或获得 `T` 的一个可变引用。与所有其他类型一样，拥有多个 `&mut UnsafeCell<T>` 别名是未定义行为。

通过使用 `UnsafeCell<T>` 作为字段，可以创建具有内部可变性的其他类型。标准库提供了几种这样的类型，这些类型都提供了安全的内部变更的 API。例如，[`std::cell::RefCell<T>`] 使用运行时借用检查来确保多个引用存在时的常规规则的执行。[`std::sync::atomic`]模块包含了一些类型，这些类型包装了一个只能通过原子操作访问的值，以允许在线程之间共享和修改该值。

[^译注1]: 当变量和指针表示的内存区域有重叠时，它们互为对方的别名。

[shared reference]: types/pointer.md#shared-references-
[ub]: behavior-considered-undefined.md
[`std::cell::UnsafeCell<T>`]: https://doc.rust-lang.org/std/cell/struct.UnsafeCell.html
[`std::cell::RefCell<T>`]: https://doc.rust-lang.org/std/cell/struct.RefCell.html
[`std::sync::atomic`]: https://doc.rust-lang.org/std/sync/atomic/index.html

<!-- 2020-11-12-->
<!-- checked -->

