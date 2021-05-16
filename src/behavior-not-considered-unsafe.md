## 不被认为是非安全的行为

>[behavior-not-considered-unsafe.md](https://github.com/rust-lang/reference/blob/master/src/behavior-not-considered-unsafe.md)\
>commit: a7473287cc6e2fb972165dc5a7ffd26dad1fc907 \
>本章译文最后维护日期：2021-1-16

虽然程序员可能（应该）发现下列行为是不良的、意外的或错误的，但 Rust 编译器并不认为这些行为是*非安全的(unsafe)*。

##### 死锁(Deadlocks)
##### 内存和其他资源的泄漏(Leaks of memory and other resources)
##### 退出而不调用析构函数(Exiting without calling destructors)
##### 通过指针泄漏暴露随机基地址(Exposing randomized base addresses through pointer leaks)
##### 整数溢出(Integer overflow)

如果程序包含算术溢出(arithmetic overflow)，则说明程序员犯了错误。在下面的讨论中，我们将区分算术溢出和包装算法(wrapping arithmetic)。前者是错误的，而后者是有意为之的。

当程序员启用了 `debug_assert!` 断言（例如，通过启用非优化的构建方式），相应的实现就必须插进来以便在溢出时触发 `panic`。而其他类型的构建形式有可能也在溢出时触发 `panics`，也有可能仅仅隐式包装一下溢出值，对溢出过程做静音处理。也就是说具体怎么对待溢出由插进来的编译实现决定。

在隐式包装溢出的情况下，（编译器实现的包装算法）实现必须通过使用二进制补码溢出(two's complement overflow)约定来提供定义良好的（即使仍然被认为是错误的）溢出包装结果。

整型提供了一些固有方法(inherent methods)，允许程序员显式地执行包装算法。例如，`i32::wrapping_add` 提供了使用二进制补码溢出约定算法的加法，即包装类加法(wrapping addition)。

标准库还提供了一个 `Wrapping<T>` 的新类型，该类型确保 `T` 的所有标准算术操作都具有包装语义。

请参阅 [RFC 560] 以了解错误条件、基本原理以及有关整数溢出的更多详细信息。

##### 逻辑错误(Logic errors)

安全代码可能会被添加一些既不能在编译时也不能在运行时检查到的逻辑限制。如果程序打破了这样的限制，其表现可能是未指定的(unspecified)，但不会导致未定义行为(undefined behavior)。这些表现可能包括 panics、错误的结果、程序中止(aborts)和程序无法终止(non-termination)。并且这些表现在运行期、构建期或各种构建期之间的的具体表现也可能有所不同。

例如，同时实现 `Hash` 和 `Eq` 就要求被认为相等的值具有相等的散列。另一个例子是像 `BinaryHeap`、`BTreeMap`、`BTreeSet`、`HashMap` 和 `HashSet` 这样的数据结构，它们描述了在数据结构中修改键的约束。违反这些约束并不被认为是非安全的，但程序（在逻辑上却）被认为是错误的，其行为是不可预测的。

[RFC 560]: https://github.com/rust-lang/rfcs/blob/master/text/0560-integer-overflow.md

<!-- 2021-1-16-->
<!-- checked -->
