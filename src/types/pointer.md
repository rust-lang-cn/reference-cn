# 指针类型

>[pointer.md](https://github.com/rust-lang/reference/blob/master/src/types/pointer.md)\
>commit: 4664361ba2f7bcc568f6bef4d119b53971fdf8ad \
>本章译文最后维护日期：2021-4-6

Rust 中所有的指针都是显式的头等(first-class)值。
它们可以被移动或复制，存储到其他数据结构中，或从函数中返回。

## 引用(`&` 和 `&mut`)

> **<sup>句法</sup>**\
> _ReferenceType_ :\
> &nbsp;&nbsp; `&` [_Lifetime_]<sup>?</sup> `mut`<sup>?</sup> [_TypeNoBounds_]

### 共享引用(`&`)

共享引用(`&`)指向*由其他值拥有的*内存。
创建了对值的共享引用可以防止对该值的直接更改。
但在某些特定情况下，[内部可变性][Interior mutability]又提供了这种情况的一种例外。
顾名思义，对一个值的共享引用的次数没有限制。共享引用类型被写为 `&type`；当需要指定显式的生存期时可写为 `&'a type`。
拷贝一个引用是一个“浅拷贝(shallow)”操作：它只涉及复制指针本身，也就是指针实现了 `Copy` trait 的意义所在。
释放引用对共享引用所指向的值没有影响，但是对[临时值][temporary value]的引用的存在将使此临时值在此引用的作用域内保持存活状态。

### 可变引用(`&mut`)

可变引用(`&mut`)也指向其他值所拥有的内存。
可变引用类型被写为 `&mut type` 或 `&'a mut type`。
可变引用（其还未被借出[^译注1]）是访问它所指向的值的唯一方法，所以可变引用没有实现 `Copy` trait。

## 裸指针(`*const` 和 `*mut`)

> **<sup>句法</sup>**\
> _RawPointerType_ :\
> &nbsp;&nbsp; `*` ( `mut` | `const` ) [_TypeNoBounds_]

裸指针是没有安全性或可用性(liveness)保证的指针。
裸指针写为 `*const T` 或 `*mut T`，例如，`*const i32` 表示指向 32-bit 有符号整数的裸指针。
拷贝或销毁(dropping )裸指针对任何其他值的生命周期(lifecycle)都没有影响。对
裸指针的解引用是[非安全(`unsafe`)操作][`unsafe` operation]，可以通过重新借用裸指针（`&*` 或 `&mut *`）将其转换为引用。
在 Rust 代码中通常不鼓励使用裸指针；它们的存在是为了提升与外部代码的互操作性，以及编写对性能要求很高的函数或很底层的函数。

在比较裸指针时，比较的是它们的地址，而不是它们指向的数据。
当比较裸指针和[动态尺寸类型][dynamically sized types]时，还会比较它们指针上的附加/元数据。

可以直接使用 [`core::ptr::addr_of!`] 创建 `*const` 类型的裸指针，通过 [`core::ptr::addr_of_mut!`] 创建 `*mut` 类型的裸指针。

## 智能指针

标准库包含了一些额外的“智能指针”类型，它们提供了在引用和裸指针这类低级指针之外的更多的功能。

[^译注1]: 译者理解这里是指 `&mut type` 如果被借出，就成了 `&&mut type`，这样就又成了不可变借用了。

[`core::ptr::addr_of!`]: ../../core/ptr/macro.addr_of.html
[`core::ptr::addr_of_mut!`]: ../../core/ptr/macro.addr_of_mut.html
[Interior mutability]: ../interior-mutability.md
[_Lifetime_]: ../trait-bounds.md
[_TypeNoBounds_]: ../types.md#type-expressions
[`unsafe` operation]: ../unsafety.md
[dynamically sized types]: ../dynamically-sized-types.md
[temporary value]: ../expressions.md#temporaries
