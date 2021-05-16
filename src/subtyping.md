# 子类型化和型变

>[subtyping.md](https://github.com/rust-lang/reference/blob/master/src/subtyping.md)\
>commit: 3b6fe80c205d2a2b5dc8a276192bbce9eeb9e9cf \
>本章译文最后维护日期：2021-03-02

子类型化是隐式的，可以出现在类型检查或类型推断的任何阶段。
Rust 中的子类型化的适用范围非常有限，仅出现在和生存期(lifetimes)的型变(variance)相关的地方，以及那些和高阶生存期相关的类型型变之间。
如果我们擦除了类型的生存期，那么唯一的子类型化就只是类型相等(type equality)了。

考虑下面的例子：字符串字面量总是拥有 `'static`生存期。不过，我们还是可以把 `s` 赋值给 `t`：

```rust
fn bar<'a>() {
    let s: &'static str = "hi";
    let t: &'a str = s;
}
```

因为 `'static` 比生存期参数 `'a` 的寿命长，所以 `&'static str` 是 `&'a str` 的子类型。

[高阶][Higher-ranked][函数指针][function pointers]和[trait对象][trait objects]可以形成另一种父子类型的关系。
它们是那些通过替换高阶生存期而得出的类型的子类型。举些例子：

```rust
// 这里 'a 被替换成了 'static
let subtype: &(for<'a> fn(&'a i32) -> &'a i32) = &((|x| x) as fn(&_) -> &_);
let supertype: &(fn(&'static i32) -> &'static i32) = subtype;

// 这对于 trait对象也是类似的
let subtype: &(for<'a> Fn(&'a i32) -> &'a i32) = &|x| x;
let supertype: &(Fn(&'static i32) -> &'static i32) = subtype;

// 我们也可以用一个高阶生存期来代替另一个
let subtype: &(for<'a, 'b> fn(&'a i32, &'b i32))= &((|x, y| {}) as fn(&_, &_));
let supertype: &for<'c> fn(&'c i32, &'c i32) = subtype;
```

## 型变

型变是泛型类型相对其参数具有的属性。
泛型类型在它的某个参数上的*型变*是描述该参数的子类型化去如何影响此泛型类型的子类型化。

* 如果 `T` 是 `U` 的一个子类型意味着 `F<T>` 是 `F<U>` 的一个子类型（即子类型化“通过(passes through)”），则 `F<T>` 在 `T` 上是*协变的(covariant)*。
* 如果 `T` 是 `U` 的一个子类型意味着 `F<U>` 是 `F<T>` 的一个子类型，则 `F<T>` 在 `T` 上是*逆变的(contravariant)*。
* 其他情况下（即不能由参数类型的子类型化关系推导出此泛型的型变关系），`F<T>` 在 `T` 上是的*不变的(invariant)*。

类型的型变关系由下表中的规则自动确定：

| Type                          | 在 `'a` 上的型变 |  在 `T` 上的型变   |
|-------------------------------|-------------------|-------------------|
| `&'a T`                       | 协变的         | 协变的         |
| `&'a mut T`                   | 协变的         | 不变的         |
| `*const T`                    |                   | 协变的         |
| `*mut T`                      |                   | 不变的         |
| `[T]` 和 `[T; n]`             |                   | 协变的         |
| `fn() -> T`                   |                   | 协变的         |
| `fn(T) -> ()`                 |                   | 逆变的     |
| `fn(T) -> T`                  |                   | 不变的         |
| `std::cell::UnsafeCell<T>`    |                   | 不变的         |
| `std::marker::PhantomData<T>` |                   | 协变的         |
| `dyn Trait<T> + 'a`           | 协变的         | 不变的         |

结构体(`struct`)、枚举(`enum`)、联合体(`union`)和元组(tuple)类型上的型变关系是通过查看其字段类型的型变关系来决定的。
如果参数用在了多处且具有不同型变关系的位置上，则该类型在该参数上是不变的。
例如，下面示例的结构体在 `'a` 和 `T` 上是协变的，在 `'b` 和 `U` 上是不变的。

```rust
use std::cell::UnsafeCell;
struct Variance<'a, 'b, T, U: 'a> {
    x: &'a U,               // 这让 `Variance` 在 'a 上是协变的, 也让在 U 上是协变的, 但是后面也使用了 U
    y: *const T,            // 在 T 上是协变的
    z: UnsafeCell<&'b f64>, // 在 'b 上是不变的
    w: *mut U,              // 在 U 上是不变的, 所以让整个结构体在 U 上是不变的
}
```

[function pointers]: types/function-pointer.md
[Higher-ranked]: https://doc.rust-lang.org/nomicon/hrtb.html
[trait objects]: types/trait-object.md
