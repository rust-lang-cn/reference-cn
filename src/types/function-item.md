# 函数项类型

>[function-item.md](https://github.com/rust-lang/reference/blob/master/src/types/function-item.md)\
>commit: b0e0ad6490d6517c19546b1023948986578fc378 \
>本章译文最后维护日期：2020-11-14

当被引用函数项、元组结构体的构造函数或枚举变体的构造函数时，会产生它们的*函数项类型(function item type)*的零尺寸的值。这种类型（也就是此值）显式地标识了该函数——标识出的内容包括程序项定义时的名字、类型参数，及其定义时的生存期参数（不是后期绑定的生存期参数，后期绑定的生存期参数只在函数被调用时才被赋与）——所以该值不需要包含一个实际的函数指针，当此函数被调用时也不需要一个间接的寻址操作去查找此函数。

没有直接引用函数项类型的句法，但是编译器会在错误消息中会显示类似于 `fn(u32) -> i32 {fn_name}` 这样的“类型”。

因为函数项类型显式地标识了其函数，所以如果它们所指的程序项不同（包括不同的程序项，或相同的程序项但泛型参数的实际类型不同），则此函数项类型也不同，混合使用它们将导致类型错误：

```rust,compile_fail,E0308
fn foo<T>() { }
let x = &mut foo::<i32>;
*x = foo::<u32>; //~ 错误：类型不匹配
```

但确实存在从函数项类型到具有相同签名的函数指针的[自动强转(coercion)][coercion]，这种自动强转一般发生在使用函数项类型却直接预期函数指针时；另一种发生情况是 `if`表达式或匹配(`match`)表达式的不同分支返回使用了具有相同签名却使用了不同函数项类型的情况：

```rust
# let want_i32 = false;
# fn foo<T>() { }

// 这里 `foo_ptr_1` 标注使用了 `fn()` 这个函数指针类型。
let foo_ptr_1: fn() = foo::<i32>;

// ... `foo_ptr_2` 也行 - 这次是通过类型检查(type-checks)做到的。
let foo_ptr_2 = if want_i32 {
    foo::<i32>
} else {
    foo::<u32>
};
```

所有的函数项类型都实现了 [`Fn`]、[`FnMut`]、[`FnOnce`]、[`Copy`]、[`Clone`]、[`Send`] 和 [`Sync`]。

[`Clone`]: ../special-types-and-traits.md#clone
[`Copy`]: ../special-types-and-traits.md#copy
[`FnMut`]: https://doc.rust-lang.org/std/ops/trait.FnMut.html
[`FnOnce`]: https://doc.rust-lang.org/std/ops/trait.FnOnce.html
[`Fn`]: https://doc.rust-lang.org/std/ops/trait.Fn.html
[`Send`]: ../special-types-and-traits.md#send
[`Sync`]: ../special-types-and-traits.md#sync
[coercion]: ../type-coercions.md
[function pointers]: function-pointer.md

<!-- 2020-11-12-->
<!-- checked -->
