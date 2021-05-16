# Trait

>[traits.md](https://github.com/rust-lang/reference/blob/master/src/items/traits.md)\
>commit: 20340ce30db8ec17b0a09dc6e07c0fa2f5c3c0ab \
>本章译文最后维护日期：2021-5-6

> **<sup>句法</sup>**\
> _Trait_ :\
> &nbsp;&nbsp; `unsafe`<sup>?</sup> `trait` [IDENTIFIER]&nbsp;
>              [_GenericParams_]<sup>?</sup>
>              ( `:` [_TypeParamBounds_]<sup>?</sup> )<sup>?</sup>
>              [_WhereClause_]<sup>?</sup> `{`\
> &nbsp;&nbsp;&nbsp;&nbsp; [_InnerAttribute_]<sup>\*</sup>\
> &nbsp;&nbsp;&nbsp;&nbsp; [_AssociatedItem_]<sup>\*</sup>\
> &nbsp;&nbsp; `}`

_trait_ 描述类型可以实现的抽象接口。这类接口由三种[关联程序项(associated items)][associated items]组成，它们分别是：

- [函数](associated-items.md#associated-functions-and-methods)
- [类型](associated-items.md#associated-types)
- [常量](associated-items.md#associated-constants)

所有 trait 都定义了一个隐式类型参数 `Self` ，它指向“实现此接口的类型”。trait 还可能包含额外的类型参数。这些类型参数，包括 `Self` 在内，都可能会[跟正常类型参数一样][generics]受到其他 trait 的约束。

trait 需要具体的类型去实现，具体的实现方法是通过该类型的各种独立实现(implementations)来完成的。

trait函数可以通过使用分号代替函数体来省略函数体。这表明此 trait的实现必须去定义实现该函数。如果 trait函数定义了一个函数体，那么这个定义就会作为任何不覆盖它的实现的默认函数实现。类似地，关联常量可以省略等号和表达式，以指示相应的实现必须定义该常量值。关联类型不能定义类型，只能在实现中指定类型。

```rust
// 有定义和没有定义的相关联trait项的例子
trait Example {
    const CONST_NO_DEFAULT: i32;
    const CONST_WITH_DEFAULT: i32 = 99;
    type TypeNoDefault;
    fn method_without_default(&self);
    fn method_with_default(&self) {}
}
```

Trait函数不能是 [`async`] 或 [`const`] 类型的。

## Trait bounds
## trait约束

泛型程序项可以使用 trait 作为其类型参数的[约束][bounds]。

## Generic Traits
## 泛型trait

可以为 trait 指定类型参数来使该 trait 成为泛型trait/泛型类型。这些类型参数出现在 trait 名称之后，使用与[泛型函数][generic functions]相同的句法。

```rust
trait Seq<T> {
    fn len(&self) -> u32;
    fn elt_at(&self, n: u32) -> T;
    fn iter<F>(&self, f: F) where F: Fn(T);
}
```

## Object Safety
## 对象安全条款

对象安全的 trait 可以是 [trait对象][trait object]的底层 trait。如果 trait 符合以下限定条件（在 [RFC 255] 中定义），则认为它是*对象安全的(object safe)*：

* 所有的超类trait[supertraits] 也必须也是对象安全的。
* 超类trait 中不能有 `Sized`。也就是说不能有 `Self: Sized`约束。
* 它必须没有任何关联常量。
* 所有关联函数必须可以从 trait对象调度分派，或者是显式不可调度分派：
    * 可调度分派函数要求：
        * 不能有类型参数（尽管生存期参数可以有）
        * 作为方法时，`Self` 只能出现在[方法][method]的接受者(receiver)的类型里，其它地方不能使用 `Self`。
        * 方法的接受者的类型必须是以下类型之一：
            * `&Self` (例如：`&self`)
            * `&mut Self` (例如：`&mut self`)
            * [`Box<Self>`]
            * [`Rc<Self>`]
            * [`Arc<Self>`]
            * [`Pin<P>`] 当 `P` 是上面类型中的一种
        * 没有 `where Self: Sized`约束（即接受者的类型 `Self`(例如：`self`) 不能有 `Sized`约束）。
    * 显式不可调度分派函数要求：
        * 有 `where Self: Sized`约束（即接受者的类型 `Self`(例如：`self`) 有 `Sized`约束）。

```rust
# use std::rc::Rc;
# use std::sync::Arc;
# use std::pin::Pin;
// 对象安全的 trait 方法。
trait TraitMethods {
    fn by_ref(self: &Self) {}
    fn by_ref_mut(self: &mut Self) {}
    fn by_box(self: Box<Self>) {}
    fn by_rc(self: Rc<Self>) {}
    fn by_arc(self: Arc<Self>) {}
    fn by_pin(self: Pin<&Self>) {}
    fn with_lifetime<'a>(self: &'a Self) {}
    fn nested_pin(self: Pin<Arc<Self>>) {}
}
# struct S;
# impl TraitMethods for S {}
# let t: Box<dyn TraitMethods> = Box::new(S);
```

```rust,compile_fail
// 此 trait 是对象安全的，但不能在 trait对象上分发(dispatch)使用这些方法。
trait NonDispatchable {
    // 非方法不能被分发。
    fn foo() where Self: Sized {}
    // 在运行之前 Self 类型未知。
    fn returns(&self) -> Self where Self: Sized;
    // `other` 可能是另一具体类型的接受者。
    fn param(&self, other: Self) where Self: Sized {}
    // 泛型与虚函数指针表(Virtual Function Pointer Table, vtable)不兼容。
    fn typed<T>(&self, x: T) where Self: Sized {}
}

struct S;
impl NonDispatchable for S {
    fn returns(&self) -> Self where Self: Sized { S }
}
let obj: Box<dyn NonDispatchable> = Box::new(S);
obj.returns(); // 错误: 不能调用带有 Self 返回类型的方法
obj.param(S);  // 错误: 不能调用带有 Self 类型的参数的方法
obj.typed(1);  // 错误: 不能调用带有泛型类型参数的方法
```

```rust,compile_fail
# use std::rc::Rc;
// 非对象安全的 trait
trait NotObjectSafe {
    const CONST: i32 = 1;  // 错误: 不能有关联常量

    fn foo() {}  // 错误: 关联函数没有 `Sized` 约束
    fn returns(&self) -> Self; // 错误: `Self` 在返回类型中 
    fn typed<T>(&self, x: T) {} // 错误: 泛型类型参数
    fn nested(self: Rc<Box<Self>>) {} // 错误: 嵌套接受者还未被完全支持。（译者注：有限支持见上面的补充规则。）
}

struct S;
impl NotObjectSafe for S {
    fn returns(&self) -> Self { S }
}
let obj: Box<dyn NotObjectSafe> = Box::new(S); // 错误
```

```rust,compile_fail
// Self: Sized trait 非对象安全的。
trait TraitWithSize where Self: Sized {}

struct S;
impl TraitWithSize for S {}
let obj: Box<dyn TraitWithSize> = Box::new(S); // 错误
```

```rust,compile_fail
// 如果有 `Self` 这样的泛型参数，那 trait 就是非对象安全的
trait Super<A> {}
trait WithSelf: Super<Self> where Self: Sized {}

struct S;
impl<A> Super<A> for S {}
impl WithSelf for S {}
let obj: Box<dyn WithSelf> = Box::new(S); // 错误: 不能使用 `Self` 作为类型参数
```

## Supertraits
## 超类trait

**超类trait** 是类型为了实现某特定 trait 而需要一并实现的 trait。此外，在任何地方，如果[泛型][generics]或 [trait对象][trait object]被某个 trait约束，那这个泛型或 trait对象就可以访问这个*超类trait* 的关联程序项。

超类trait 是通过 trait 的 `Self`类型上的 trait约束来声明的，并且通过这种声明 trait约束的方式来传递这种超类trait 关系。一个 trait 不能是它自己的超类trait。

有超类trait 的 trait 称其为其超类trait 的**子trait(subtrait)**。

下面是一个声明 `Shape` 是 `Circle` 的超类trait 的例子。

```rust
trait Shape { fn area(&self) -> f64; }
trait Circle : Shape { fn radius(&self) -> f64; }
```

下面是同一个示例，除了改成使用 [where子句][where clauses]来等效实现。

```rust
trait Shape { fn area(&self) -> f64; }
trait Circle where Self: Shape { fn radius(&self) -> f64; }
```

下面例子通过 `Shape` 的 `area` 函数为 `radius` 提供了一个默认实现：

```rust
# trait Shape { fn area(&self) -> f64; }
trait Circle where Self: Shape {
    fn radius(&self) -> f64 {
        // 因为 A = pi * r^2，所以通过代数推导得：r = sqrt(A / pi)
        (self.area() /std::f64::consts::PI).sqrt()
    }
}
```

下一个示例调用了一个泛型参数的超类trait 上的方法。

```rust
# trait Shape { fn area(&self) -> f64; }
# trait Circle : Shape { fn radius(&self) -> f64; }
fn print_area_and_radius<C: Circle>(c: C) {
    // 这里我们调用 `Circle` 的超类trait 的 area 方法。
    println!("Area: {}", c.area());
    println!("Radius: {}", c.radius());
}
```

类似地，这里是一个在 trait对象上调用超类trait 的方法的例子。

```rust
# trait Shape { fn area(&self) -> f64; }
# trait Circle : Shape { fn radius(&self) -> f64; }
# struct UnitCircle;
# impl Shape for UnitCircle { fn area(&self) -> f64 { std::f64::consts::PI } }
# impl Circle for UnitCircle { fn radius(&self) -> f64 { 1.0 } }
# let circle = UnitCircle;
let circle = Box::new(circle) as Box<dyn Circle>;
let nonsense = circle.radius() * circle.area();
```

## Unsafe traits
## 非安全trait

以关键字 `unsafe` 开头的 trait程序项表示*实现*该 trait 可能是[非安全][unsafe]的。使用正确实现的非安全trait 是安全的。[trait实现][trait implementation]也必须以关键字 `unsafe` 开头。

[`Sync`] 和 [`Send`] 是典型的非安全trait。

## Parameter patterns
## 参数模式

（trait 中）没有代码体的函数声明或方法声明（的参数模式）只允许使用[标识符/IDENTIFIER][IDENTIFIER]模式 或 `_` [通配符][WildcardPattern]模式。当前 `mut` [IDENTIFIER] 还是允许的，但已被弃用，未来将成为一个硬编码错误(hard error)。
<!-- https://github.com/rust-lang/rust/issues/35203 -->

在 2015 版中，trait 的函数或方法的参数模式是可选的：

```rust
trait T {
    fn f(i32);  // 不需要参数的标识符.
}
```

所有的参数模式被限制为下述之一：

* [IDENTIFIER]
* `mut` [IDENTIFIER]
* [`_`][WildcardPattern]
* `&` [IDENTIFIER]
* `&&` [IDENTIFIER]

（跟普通函数一样，）从 2018 版开始，（trait 中）函数或方法的参数模式不再是可选的。同时，也跟普通函数一样，（trait 中）函数或方法只要有代码体，其参数模式可以是任何不可反驳型模式。但如果没有代码体，上面列出的限制仍然有效。

```rust,edition2018
trait T {
    fn f1((a, b): (i32, i32)) {}
    fn f2(_: (i32, i32));  // 没有代码体不能使用元组模式。
}
```

## Item visibility
## 程序项的可见性

依照句法规定，trait程序项在语法上允许使用 [_Visibility_]句法的注释，但是当 trait 被（句法法分析程序）验证(validate)后，该可见性注释又被弃用。因此，在源码解析层面，可以在使用程序项的不同上下文中使用统一的语法对这些程序项进行解析。例如，空的 `vis` 宏匹配段选择器可以用于 trait程序项，而在其他允许使用非空可见性的情况下，也可使用这同一套宏规则。

```rust
macro_rules! create_method {
    ($vis:vis $name:ident) => {
        $vis fn $name(&self) {}
    };
}

trait T1 {
    // 只允许空 `vis`。
    create_method! { method_of_t1 }
}

struct S;

impl S {
    // 这里允许使用非空可见性。
    create_method! { pub method_of_s }
}

impl T1 for S {}

fn main() {
    let s = S;
    s.method_of_t1();
    s.method_of_s();
}
```

[^译者备注]: 两点提醒：所有 trait 都定义了一个隐式类型参数 `Self` ，它指向“实现此接口的类型”；trait 的 Self 默认满足：`Self: ?Sized`

[IDENTIFIER]: ../identifiers.md
[WildcardPattern]: ../patterns.md#wildcard-pattern
[_AssociatedItem_]: associated-items.md
[_GenericParams_]: generics.md
[_InnerAttribute_]: ../attributes.md
[_TypeParamBounds_]: ../trait-bounds.md
[_Visibility_]: ../visibility-and-privacy.md
[_WhereClause_]: generics.md#where-clauses
[bounds]: ../trait-bounds.md
[trait object]: ../types/trait-object.md
[RFC 255]: https://github.com/rust-lang/rfcs/blob/master/text/0255-object-safety.md
[associated items]: associated-items.md
[method]: associated-items.md#methods
[supertraits]: #supertraits
[implementations]: implementations.md
[generics]: generics.md
[where clauses]: generics.md#where-clauses
[generic functions]: functions.md#generic-functions
[unsafe]: ../unsafety.md
[trait implementation]: implementations.md#trait-implementations
[`Send`]: ../special-types-and-traits.md#send
[`Sync`]: ../special-types-and-traits.md#sync
[`Arc<Self>`]: ../special-types-and-traits.md#arct
[`Box<Self>`]: ../special-types-and-traits.md#boxt
[`Pin<P>`]: ../special-types-and-traits.md#pinp
[`Rc<Self>`]: ../special-types-and-traits.md#rct
[`async`]: functions.md#async-functions
[`const`]: functions.md#const-functions
