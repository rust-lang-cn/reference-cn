# 特殊类型和 trait

>[special-types-and-traits.md](https://github.com/rust-lang/reference/blob/master/src/special-types-and-traits.md)\
>commit: fcdc0cab546c10921d66054be25c6afc9dd6b3bc \
>本章译文最后维护日期：2020-11-16

[标准库][the standard library]中的某些类型和 trait 在 Rust 编译器中也直接能用。本章就阐述了这些类型和 trait 的特殊特性。

## `Box<T>`

[`Box<T>`] 有一些特殊的特性，Rust 目前还不允许用户定义类型时使用。

* `Box<T>` 的[解引用操作符]会产生一个可从中移出值的内存位置[^译注1]。这（种特殊性）意味着应用在 `Box<T>` 上的 `*`运算符和 `Box<T>` 的析构函数都是语言内置的。
* [方法][Methods]可以使用 `Box<Self>` 作为接受者。
* `Box<T>` 可以绕过[孤儿规则(orphan rules)][orphan rules]，与 `T` 在同一 crate 中实现同一 trait，其他泛型类型无法绕过。

## `Rc<T>`

[方法][Methods]可以使用 [`Rc<Self>`] 作为接受者。

## `Arc<T>`

[方法][Methods]可以使用 [`Arc<Self>`] 作为接受者。

## `Pin<P>`

[方法][Methods]可以使用 [`Pin<P>`] 作为接受者。

## `UnsafeCell<T>`

[`std::cell::UnsafeCell<T>`] 用于[内部可变性][interior mutability]。它确保编译器不会对此类类型执行不正确的优化。它还能确保具有内部可变类型的[静态(`static`)项][`static` items]不会被放在标记为只读的内存中。

## `PhantomData<T>`

[`std::marker::PhantomData<T>`] 是一个零尺寸零的、最小对齐量的、被认为拥有(own) `T` 的类型，这个类型存在目的是应用在确定[型变][variance]关系、[销毁检查][drop check]和[自动trait](#auto-traits) 中的。

## Operator Traits
## 运算符/操作符trait

[`std::ops`] 和 [`std::cmp`] 中的 trait 看用于重载 Rust 的[运算符/操作符][operators]、[索引表达式][indexing expressions]和[调用表达式][call expressions]。

## `Deref` and `DerefMut`

除了重载一元 `*`运算符外，[`Deref`] 和 [`DerefMut`] 也用于[方法解析(method resolution)][method resolution]和[利用 `Deref` 达成自动强转][deref coercions]。

## `Drop`

[`Drop`] trait 提供了一个[析构函数][destructor]，每当要销毁此类型的值时就会运行它。

## `Copy`

[`Copy`] trait 改变实现它的类型的语义。其类型实现了 `Copy` 的值在赋值时将被复制而不是移动。

只能为未实现 `Drop` trait 且字段都是 `Copy` 的类型实现 `Copy`。[^译注2]\
对于枚举，这意味着所有变体的所有字段都必须是 `Copy` 的。\
对于联合体，这意味着所有的变体都必须是 `Copy` 的。

`Copy` 已由编译器实现给了下述类型：

* [数字类类型][Numeric types]
* `char`, `bool`, 和 [`!`]
* 由 `Copy` 类型的元素组成的[元组][Tuples]
* 由 `Copy` 类型的元素组成的[数组][Arrays]
* [共享引用][Shared references]
* [裸指针][Raw pointers]
* [函数指针][Function pointers] 和 [函数项类型][function item types]

## `Clone`

[`Clone`] trait 是 `Copy` 的超类trait，所以它也需要编译器生成实现。它被编译器实现给了以下类型：

* 实现了内置的 `Copy` trait 的类型（见上面）
* 由 `Clone` 类型的元素组成的[元组][Tuples]
* 由 `Clone` 类型的元素组成的[数组][Arrays]

## `Send`

[`Send`] trait 表明这种类型的值可以安全地从一个线程发送给另一个线程。

## `Sync`

[`Sync`] trait 表示在多个线程之间共享这种类型的值是安全的。必须为不可变[静态(`static`)项][`static` items]中使用的所有类型实现此 trait。

## Auto traits

[`Send`]、[`Sync`]、[`Unpin`]、[`UnwindSafe`] 和 [`RefUnwindSafe`] trait 都是*自动trait(auto traits)*。自动trait 具有特殊的属性。

对于给定类型，如果没有为其显式实现或否定实现(negative implementation)某自动trait，那么编译器就会根据以下规则去自动此为类型实现该自动trait：

* 如果 `T` 实现了某自动trait，那 `&T`、`&mut T`、`*const T`、`*mut T`、`[T; n]` 和 `[T]` 也会实现此自动trait。
* 函数项类型(function item types)和函数指针会自动实现这些自动trait。
* 如果结构体、枚举、联合体和元组它们的所有字段都实现了这些自动trait，则它们本身也会自动实现这些自动trait。
* 如果闭包捕获的所有变量的类型都实现了这些自动trait，那么闭包会自动实现这些自动trait。一个闭包通过共享引用捕获了一个 `T`，同时通过传值的方式捕获了一个 `U`，那么该闭包会自动实现 `&T` 和 `U` 所共同实现的那些自动trait。

对于泛型类型（上面的这些内置类型也算是建立在 `T` 上的泛型），如果泛型实现在当前已够用，则编译器不会再为其实现其他的自动trait，除非它们不满足必需的 trait约束。例如，标准库在 `T` 是 `Sync` 的地方都为 `&T` 实现了 `Send`；这意味着如果 `T` 是 `Send`，而不是 `Sync`，编译器就不会为 `&T` 自动实现 `Send`。

自动trait 也可以有否定实现，在标准库文档中表示为 `impl !AutoTrait for T`，它覆盖了自动实现。例如，`*mut T` 有一个关于 `Send` 的否定实现，所以 `*mut T` 不是 `Send` 的，即使 `T` 是。目前还没有稳下来的方法来指定其他类型的否定实现；目前否定实现只能在标准库里可以稳定使用。[^译注3]

自动trait 可以作为额外的约束添加到任何 [trait对象][trait object]上，尽管通常我们见到的 trait对象的类型名上一般只显示一个 trait。例如，`Box<dyn Debug + Send + UnwindSafe>` 就是一个有效的类型。

## `Sized`

[`Sized`] trait表明这种类型的尺寸在编译时是已知的；也就是说，它不是一个[动态尺寸类型][dynamically sized type]。[类型参数][Type parameters]默认是 `Sized` 的。`Sized` 总是由编译器自动实现，而不是由[实现(implementation items)][implementation items]主动实现的。

[^译注1]: 这里是相对普通的借用/引用来说，普通的借用/引用对指向的内存位置不拥有所有权，所以无法从中移出值。

[^译注2]: 实现 `Drop` trait 的类型只能是使用移动语义(move)的类型。

[^译注3]: 标准库外使用需要打开特性 `#![feature(negative_impls)]`。

[`Arc<Self>`]: https://doc.rust-lang.org/std/sync/struct.Arc.html
[`Box<T>`]: https://doc.rust-lang.org/std/boxed/struct.Box.html
[`Clone`]: https://doc.rust-lang.org/std/clone/trait.Clone.html
[`Copy`]: https://doc.rust-lang.org/std/marker/trait.Copy.html
[`Deref`]: https://doc.rust-lang.org/std/ops/trait.Deref.html
[`DerefMut`]: https://doc.rust-lang.org/std/ops/trait.DerefMut.html
[`Drop`]: https://doc.rust-lang.org/std/ops/trait.Drop.html
[`Pin<P>`]: https://doc.rust-lang.org/std/pin/struct.Pin.html
[`Rc<Self>`]: https://doc.rust-lang.org/std/rc/struct.Rc.html
[`RefUnwindSafe`]: https://doc.rust-lang.org/std/panic/trait.RefUnwindSafe.html
[`Send`]: https://doc.rust-lang.org/std/marker/trait.Send.html
[`Sized`]: https://doc.rust-lang.org/std/marker/trait.Sized.html
[`std::cell::UnsafeCell<T>`]: https://doc.rust-lang.org/std/cell/struct.UnsafeCell.html
[`std::cmp`]: https://doc.rust-lang.org/std/cmp/index.html
[`std::marker::PhantomData<T>`]: https://doc.rust-lang.org/std/marker/struct.PhantomData.html
[`std::ops`]: https://doc.rust-lang.org/std/ops/index.html
[`UnwindSafe`]: https://doc.rust-lang.org/std/panic/trait.UnwindSafe.html
[`Sync`]: https://doc.rust-lang.org/std/marker/trait.Sync.html
[`Unpin`]: https://doc.rust-lang.org/std/marker/trait.Unpin.html

[Arrays]: types/array.md
[call expressions]: expressions/call-expr.md
[deref coercions]: type-coercions.md#coercion-types
[dereference operator]: expressions/operator-expr.md#the-dereference-operator
[destructor]: destructors.md
[drop check]: https://doc.rust-lang.org/nomicon/dropck.html
[dynamically sized type]: dynamically-sized-types.md
[Function pointers]: types/function-pointer.md
[function item types]: types/function-item.md
[implementation items]: items/implementations.md
[indexing expressions]: expressions/array-expr.md#array-and-slice-indexing-expressions
[interior mutability]: interior-mutability.md
[Numeric types]: types/numeric.md
[Methods]: items/associated-items.md#associated-functions-and-methods
[method resolution]: expressions/method-call-expr.md
[operators]: expressions/operator-expr.md
[orphan rules]: items/implementations.md#trait-implementation-coherence
[Raw pointers]: types/pointer.md#raw-pointers-const-and-mut
[`static` items]: items/static-items.md
[Shared references]: types/pointer.md#shared-references-
[the standard library]: https://doc.rust-lang.org/std/index.html
[trait object]: types/trait-object.md
[Tuples]: types/tuple.md
[Type parameters]: types/parameters.md
[variance]: subtyping.md#variance
[`!`]: types/never.md

<!-- 2020-11-12-->
<!-- checked -->
