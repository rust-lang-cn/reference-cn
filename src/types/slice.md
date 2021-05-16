# 切片类型

>[slice.md](https://github.com/rust-lang/reference/blob/master/src/types/slice.md)\
>commit: b0e0ad6490d6517c19546b1023948986578fc378 \
>本章译文最后维护日期：2020-11-14

> **<sup>句法</sup>**\
> _SliceType_ :\
> &nbsp;&nbsp; `[` [_Type_] `]`

切片是一种[动态尺寸类型(dynamically sized type)][dynamically sized type]，它代表类型为 `T` 的元素组成的数据序列的一个“视图(view)”。切片类型写为 `[T]`。

要使用切片类型，通常必须放在指针后面使用，例如：

* `&[T]`，共享切片('shared slice')，常被直接称为切片(`slice`)，它不拥有它指向的数据，只是借用。
* `&mut [T]`，可变切片('mutable slice')，可变借用它指向的数据。
* `Box<[T]>`, boxed切片('boxed slice')。

示例：

```rust
// 一个堆分配的数组，被自动强转成切片
let boxed_array: Box<[i32]> = Box::new([1, 2, 3]);

// 数组上的（共享）切片
let slice: &[i32] = &boxed_array[..];
```

切片的所有元素总是初始化过的，使用 Rust 中的安全(safe)方法或操作符来访问切片时总是会做越界检查。

[_Type_]: ../types.md#type-expressions
[dynamically sized type]: ../dynamically-sized-types.md

<!-- 2020-11-12-->
<!-- checked -->
