# 关联项

>[associated-items.md](https://github.com/rust-lang/reference/blob/master/src/items/associated-items.md)\
>commit: 761ad774fcb300f2b506fed7b4dbe753cda88d80 \
>本章译文最后维护日期：2021-1-17

> **<sup>句法</sup>**\
> _AssociatedItem_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> (\
> &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; [_MacroInvocationSemi_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | ( [_Visibility_]<sup>?</sup> ( [_TypeAlias_] | [_ConstantItem_] | [_Function_] ) )\
> &nbsp;&nbsp; )


*关联程序项*是在 [traits] 中声明或在[实现][implementations]中定义的程序项。之所以这样称呼它们，是因为它们是被定义在一个相关联的类型（即实现里指定的类型）上的。关联程序项是那些可在模块中声明的程序项的子集。具体来说，有[关联函数][associated functions]（包括方法）、[关联类型][associated types]和[关联常量][associated constants]。

[关联函数]: #associated-functions-and-methods
[关联类型]: #associated-types
[关联常量]: #associated-constants

当关联程序项与被关联程序项在逻辑上相关时，关联程序项就非常有用。例如，`Option` 上的 `is_some` 方法内在逻辑定义上就与 Option枚举类型相关，所以它应该和 Option 关联在一起。

每个关联程序项都有两种形式：（包含实际实现的）定义和（为定义声明签名的）声明。[^译者备注1]

正是这些声明构成了 trait 的契约(contract)以及其泛型参数中的可用内容。

## 关联函数和方法

*关联函数*是与一个类型相关联的[函数][functions]。

*关联函数声明*为*关联函数定义*声明签名。它的书写格式和函数项一样，除了函数体被替换为 `;`。

标识符是关联函数的名称。关联函数的泛型参数、参数列表、返回类型 和 where子句必须与它们在*关联函数声明*中声明的格式一致。

*关联函数定义*定义与另一个类型相关联的函数。它的编写方式与[函数项][function item]相同。

常见的关联函数的一个例子是 `new`函数，它返回此关联函数所关联的类型的值。

```rust
struct Struct {
    field: i32
}

impl Struct {
    fn new() -> Struct {
        Struct {
            field: 0i32
        }
    }
}

fn main () {
    let _struct = Struct::new();
}
```

当关联函数在 trait 上声明时，此函数也可以通过一个指向 trait，再后跟函数名的[路径][path]来调用。当发生这种情况时，可以用 trait 的实际路径和关联函数的标识符按 `<_ as Trait>::function_name` 这样的形式来组织实际的调用路径。

```rust
trait Num {
    fn from_i32(n: i32) -> Self;
}

impl Num for f64 {
    fn from_i32(n: i32) -> f64 { n as f64 }
}

// 在这个案例中，这4种形式都是等价的。
let _: f64 = Num::from_i32(42);
let _: f64 = <_ as Num>::from_i32(42);
let _: f64 = <f64 as Num>::from_i32(42);
let _: f64 = f64::from_i32(42);
```

### 方法

如果关联函数的参数列表中的第一个参数名为 `self`[^译者备注2]，则此关联函数被称为*方法*，方法可以使用[方法调用操作符][method call operator](`.`)来调用，例如 `x.foo()`，也可以使用常用的函数调用形式进行调用。

如果名为 `self` 的参数的类型被指定了，它就通过以下文法（其中 `'lt` 表示生存期参数）来把此指定的参数限制解析成此文法中的一个类型：

```text
P = &'lt S | &'lt mut S | Box<S> | Rc<S> | Arc<S> | Pin<P>
S = Self | P
```

此文法中的 `Self`终结符(terminal)表示解析为实现类型(implementing type)的类型。这种解析包括解析上下文中的类型别名 `Self`、其他类型别名、或使用投射解析把（self 的类型中的）关联类型解析为实现类型。[^译者备注3]

> 译者注：原谅译者对上面这句背后知识的模糊理解，那首先给出原文：\
> The `Self` terminal in this grammar denotes a type resolving to the implementing type. This can also include the contextual type alias `Self`, other type aliases, or associated type projections resolving to the implementing type. \
> 译者在此先邀请读者中的高手帮忙翻译清楚。感谢感谢。另外译者还是要啰嗦以下译者对这句话背后知识的理解，希望有人能指出其中的错误，以让译者有机会进步：\
> 首先终结符(terminal)就是不能再更细分的词法单元，可以理解它是一个 token，这里它代表 self（即方法接受者）的类型的基础类型。上面句法中的 P 代表一个产生式，它内部定义的规则是并联的，就是自动机在应用这个产生式时碰到任意符合条件的输入就直接进入终态。S 代表有限自动机从 S 这里开始读取 self 的类型。这里 S 是 Self 和 P 的并联，应该表示是：如果 self 的类型直接是 Self，那就直接进入终态，即返回 Self，即方法接收者的直接类型就是结果类型；如果 self 的类型是 P 中的任一种，就返回那一种，比如 self 的类型是一个 box指针，那么就返回 `Box<S>`。


```rust
# use std::rc::Rc;
# use std::sync::Arc;
# use std::pin::Pin;
// 结构体 `Example` 上的方法示例
struct Example;
type Alias = Example;
trait Trait { type Output; }
impl Trait for Example { type Output = Example; }
impl Example {
    fn by_value(self: Self) {}
    fn by_ref(self: &Self) {}
    fn by_ref_mut(self: &mut Self) {}
    fn by_box(self: Box<Self>) {}
    fn by_rc(self: Rc<Self>) {}
    fn by_arc(self: Arc<Self>) {}
    fn by_pin(self: Pin<&Self>) {}
    fn explicit_type(self: Arc<Example>) {}
    fn with_lifetime<'a>(self: &'a Self) {}
    fn nested<'a>(self: &mut &'a Arc<Rc<Box<Alias>>>) {}
    fn via_projection(self: <Example as Trait>::Output) {}
}
```

（方法的首参）可以在不指定类型的情况下使用简写句法，具体对比如下：

简写模式               | 等效项
----------------------|-----------
`self`                | `self: Self`
`&'lifetime self`     | `self: &'lifetime Self`
`&'lifetime mut self` | `self: &'lifetime mut Self`

> **注意**: （方法的）生存期也能，其实也经常是使用这种方式来省略。

如果 `self` 参数以 `mut` 为前缀，它就变成了一个可变的变量，类似于使用 `mut` [标识符模式][identifier pattern]的常规参数。例如：

```rust
trait Changer: Sized {
    fn change(mut self) {}
    fn modify(mut self: Box<Self>) {}
}
```

以下是一个关于 trait 的方法的例子，现给定如下内容：

```rust
# type Surface = i32;
# type BoundingBox = i32;
trait Shape {
    fn draw(&self, surface: Surface);
    fn bounding_box(&self) -> BoundingBox;
}
```

这里定义了一个带有两个方法的 trait。当此 trait 被引入当前作用域内后，所有此 trait 的[实现][implementations]的值都可以调用此 trait 的 `draw` 和 `bounding_box` 方法。

```rust
# type Surface = i32;
# type BoundingBox = i32;
# trait Shape {
#     fn draw(&self, surface: Surface);
#     fn bounding_box(&self) -> BoundingBox;
# }
#
struct Circle {
    // ...
}

impl Shape for Circle {
    // ...
#   fn draw(&self, _: Surface) {}
#   fn bounding_box(&self) -> BoundingBox { 0i32 }
}

# impl Circle {
#     fn new() -> Circle { Circle{} }
# }
#
let circle_shape = Circle::new();
let bounding_box = circle_shape.bounding_box();
```

> **版本差异**: 在 2015 版中, 使用匿名参数来声明 trait方法是可能的 (例如：`fn foo(u8)`)。在 2018 版本中，这已被弃用，再用会导致编译错误。新版本种所有的参数都必须有参数名。

#### 方法参数上的属性

方法参数上的属性遵循与[常规函数参数][regular function parameters]上相同的规则和限制。

## 关联类型

*关联类型*是与另一个类型关联的[类型别名(type aliases)][type aliases] 。关联类型不能在[固有实现][inherent implementations]中定义，也不能在 trait 中给它们一个默认实现。

*关联类型声明*为*关联类型定义*声明签名。书写形式为：先是 `type`，然后是一个[标识符][identifier]，最后是一个可选的 trait约束列表。

这里的标识符是声明的类型的别名名称；可选的 trait约束必须由此类型别名的实现来履行实现。

*关联类型定义*在另一个类型上定义了一个类型别名。书写形式为：先是 `type`，然后是一个[标识符]，然后再是一个 `=`，最后是一个[类型][type]。

如果类型 `Item` 上有一个来自 trait `Trait`的关联类型 `Assoc`，则表达式 `<Item as Trait>::Assoc` 也是一个类型，具体就是*关联类型定义*中指定的类型的一个别名。此外，如果 `Item` 是类型参数，则 `Item::Assoc` 也可以在类型参数中使用。

关联类型不能包括[泛型参数][generic parameters]或 [where子句][where clauses]。

```rust
trait AssociatedType {
    // 关联类型声明
    type Assoc;
}

struct Struct;

struct OtherStruct;

impl AssociatedType for Struct {
    // 关联类型定义
    type Assoc = OtherStruct;
}

impl OtherStruct {
    fn new() -> OtherStruct {
        OtherStruct
    }
}

fn main() {
    // 使用 <Struct as AssociatedType>::Assoc 来引用关联类型 OtherStruct
    let _other_struct: OtherStruct = <Struct as AssociatedType>::Assoc::new();
}
```

### 示例展示容器内的关联类型

下面给出一个 `Container` trait 示例。请注意，该类型可用在方法签名内：

```rust
trait Container {
    type E;
    fn empty() -> Self;
    fn insert(&mut self, elem: Self::E);
}
```

为了能让实现类型来实现此 trait，实现类型不仅必须为每个方法提供实现，而且必须指定类型 `E`。下面是一个为标准库类型 `Vec` 实现了此 `Container` 的实现：

```rust
# trait Container {
#     type E;
#     fn empty() -> Self;
#     fn insert(&mut self, elem: Self::E);
# }
impl<T> Container for Vec<T> {
    type E = T;
    fn empty() -> Vec<T> { Vec::new() }
    fn insert(&mut self, x: T) { self.push(x); }
}
```

## 关联常量

*关联常量*是与具体类型关联的[常量][constants]。

*关联常量声明*为*关联常量定义*声明签名。书写形式为：先是 `const` 开头，然后是标识符，然后是 `:`，然后是一个类型，最后是一个 `;`。

这里标识符是（外部引用）路径中使用的常量的名称；类型是（此关联常量的）定义必须实现的类型。

*关联常量定义*定义了与类型关联的常量。它的书写方式与[常量项][constant item]相同。

### 示例展示关联常量

基本示例：

```rust
trait ConstantId {
    const ID: i32;
}

struct Struct;

impl ConstantId for Struct {
    const ID: i32 = 1;
}

fn main() {
    assert_eq!(1, Struct::ID);
}
```

使用默认值：

```rust
trait ConstantIdDefault {
    const ID: i32 = 1;
}

struct Struct;
struct OtherStruct;

impl ConstantIdDefault for Struct {}

impl ConstantIdDefault for OtherStruct {
    const ID: i32 = 5;
}

fn main() {
    assert_eq!(1, Struct::ID);
    assert_eq!(5, OtherStruct::ID);
}
```

[^译者备注1]: 固有实现中声明和定义是在一起的。

[^译者备注2]: 把简写形式转换成等价的标准形式。

[^译者备注3]: 结合下面的示例理解。

[_ConstantItem_]: constant-items.md
[_Function_]: functions.md
[_MacroInvocationSemi_]: ../macros.md#macro-invocation
[_OuterAttribute_]: ../attributes.md
[_TypeAlias_]: type-aliases.md
[_Visibility_]: ../visibility-and-privacy.md
[`Arc<Self>`]: ../special-types-and-traits.md#arct
[`Box<Self>`]: ../special-types-and-traits.md#boxt
[`Pin<P>`]: ../special-types-and-traits.md#pinp
[`Rc<Self>`]: ../special-types-and-traits.md#rct
[traits]: traits.md
[type aliases]: type-aliases.md
[inherent implementations]: implementations.md#inherent-implementations
[identifier]: ../identifiers.md
[identifier pattern]: ../patterns.md#identifier-patterns
[implementations]: implementations.md
[type]: ../types.md#type-expressions
[constants]: constant-items.md
[constant item]: constant-items.md
[functions]: functions.md
[function item]: ../types/function-item.md
[method call operator]: ../expressions/method-call-expr.md
[path]: ../paths.md
[regular function parameters]: functions.md#attributes-on-function-parameters
[generic parameters]: generics.md
[where clauses]: generics.md#where-clauses

<!-- 2021-1-17-->
<!-- checked -->
