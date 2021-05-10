# 路径

_路径_ 是一个或多个由<span class="parenthetical">命名空间限定符（`::`）</span>_逻辑_ 分隔的路径片段序列。如果路径仅由一个路径片段组成，则它引用当前控制域内的[项][item]或[变量][variable]。如果路径包含多个路径片段，则常是引用具体项。

两个仅由标识符部分组成的简单路径的例子：

<!-- ignore: syntax fragment -->
```rust,ignore
x;
x::y::z;
```

## 路径类型

### 简单路径

> **<sup>Syntax</sup>**\
> _SimplePath_ :\
> &nbsp;&nbsp; `::`<sup>?</sup> _SimplePathSegment_ (`::` _SimplePathSegment_)<sup>\*</sup>
>
> _SimplePathSegment_ :\
> &nbsp;&nbsp; [IDENTIFIER] | `super` | `self` | `crate` | `$crate`

简单路径常用于[可见性][visibility]标记、[属性][attributes]、[声明宏][macros]，以及 [`use`] 声明项。例如：

```rust
use std::io::{self, Write};
mod m {
    #[clippy::cyclomatic_complexity = "0"]
    pub (in super) fn f1() {}
}
```

### 表达式路径

> **<sup>Syntax</sup>**\
> _PathInExpression_ :\
> &nbsp;&nbsp; `::`<sup>?</sup> _PathExprSegment_ (`::` _PathExprSegment_)<sup>\*</sup>
>
> _PathExprSegment_ :\
> &nbsp;&nbsp; _PathIdentSegment_ (`::` _GenericArgs_)<sup>?</sup>
>
> _PathIdentSegment_ :\
> &nbsp;&nbsp; [IDENTIFIER] | `super` | `self` | `Self` | `crate` | `$crate`
>
> _GenericArgs_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `<` `>`\
> &nbsp;&nbsp; | `<` _GenericArgsLifetimes_ `,`<sup>?</sup> `>`\
> &nbsp;&nbsp; | `<` _GenericArgsTypes_ `,`<sup>?</sup> `>`\
> &nbsp;&nbsp; | `<` _GenericArgsBindings_ `,`<sup>?</sup> `>`\
> &nbsp;&nbsp; | `<` _GenericArgsTypes_ `,` _GenericArgsBindings_ `,`<sup>?</sup> `>`\
> &nbsp;&nbsp; | `<` _GenericArgsLifetimes_ `,` _GenericArgsTypes_ `,`<sup>?</sup> `>`\
> &nbsp;&nbsp; | `<` _GenericArgsLifetimes_ `,` _GenericArgsBindings_ `,`<sup>?</sup> `>`\
> &nbsp;&nbsp; | `<` _GenericArgsLifetimes_ `,` _GenericArgsTypes_ `,` _GenericArgsBindings_ `,`<sup>?</sup> `>`
>
> _GenericArgsLifetimes_ :\
> &nbsp;&nbsp; [_Lifetime_] (`,` [_Lifetime_])<sup>\*</sup>
>
> _GenericArgsTypes_ :\
> &nbsp;&nbsp; [_Type_] (`,` [_Type_])<sup>\*</sup>
>
> _GenericArgsBindings_ :\
> &nbsp;&nbsp; _GenericArgsBinding_ (`,` _GenericArgsBinding_)<sup>\*</sup>
>
> _GenericArgsBinding_ :\
> &nbsp;&nbsp; [IDENTIFIER] `=` [_Type_]

表达式路径允许为路径指定泛型参数，被用在各种[表达式][expressions]和[模式][patterns]中。

记号 `::` 出现在泛型参数开端的尖括号 `<` 之前，以避免小于号的歧义，俗称“涡轮鱼（turbofish）”语法。

```rust
(0..10).collect::<Vec<_>>();
Vec::<u8>::with_capacity(1024);
```

## 限定路径

> **<sup>Syntax</sup>**\
> _QualifiedPathInExpression_ :\
> &nbsp;&nbsp; _QualifiedPathType_ (`::` _PathExprSegment_)<sup>+</sup>
>
> _QualifiedPathType_ :\
> &nbsp;&nbsp; `<` [_Type_] (`as` _TypePath_)? `>`
>
> _QualifiedPathInType_ :\
> &nbsp;&nbsp; _QualifiedPathType_ (`::` _TypePathSegment_)<sup>+</sup>

完全限定的路径允许消除 [trait 实现][trait implementations]的路径歧义和指定[规范路径](#规范路径)，在使用时支持类型语法。详述如下：

```rust
struct S;
impl S {
    fn f() { println!("S"); }
}
trait T1 {
    fn f() { println!("T1 f"); }
}
impl T1 for S {}
trait T2 {
    fn f() { println!("T2 f"); }
}
impl T2 for S {}
S::f();  // Calls the inherent impl.
<S as T1>::f();  // Calls the T1 trait function.
<S as T2>::f();  // Calls the T2 trait function.
```

### 类型路径

> **<sup>Syntax</sup>**\
> _TypePath_ :\
> &nbsp;&nbsp; `::`<sup>?</sup> _TypePathSegment_ (`::` _TypePathSegment_)<sup>\*</sup>
>
> _TypePathSegment_ :\
> &nbsp;&nbsp; _PathIdentSegment_ `::`<sup>?</sup> ([_GenericArgs_] | _TypePathFn_)<sup>?</sup>
>
> _TypePathFn_ :\
> `(` _TypePathFnInputs_<sup>?</sup> `)` (`->` [_Type_])<sup>?</sup>
>
> _TypePathFnInputs_ :\
> [_Type_] (`,` [_Type_])<sup>\*</sup> `,`<sup>?</sup>

类型路径用于类型定义、trait 约束、类型参数约束，以及限定路径。

尽管在泛型参数之前允许使用记号 `::`，但其不是必需的。因为在类型路径中，没有如同限定路径中 _PathInExpression_ 那样的歧义。

```rust
# mod ops {
#     pub struct Range<T> {f1: T}
#     pub trait Index<T> {}
#     pub struct Example<'a> {f1: &'a i32}
# }
# struct S;
impl ops::Index<ops::Range<usize>> for S { /*...*/ }
fn i<'a>() -> impl Iterator<Item = ops::Example<'a>> {
    // ...
#    const EXAMPLE: Vec<ops::Example<'static>> = Vec::new();
#    EXAMPLE.into_iter()
}
type G = std::boxed::Box<dyn std::ops::FnOnce(isize) -> isize>;
```

## 路径限定符

路径可以通过多种前导限定符来改变其解析的方式。

### `::`

以 `::` 开头的路径被认为是全局路径，路径段从 crate 根位置开始解析。路径中的每个标识符都必须被解析为一个项。

> **版本差异**：2015 版本中，crate 根包含多种不同的项，包括：外部 crate、默认 crate（如 `std`、`core`），并且各项可以用作 crate（包括 `use`）最高级别。
>
> 从 2018 版本始，以 `::` 开头的路径仅能引用crate。

```rust
mod a {
    pub fn foo() {}
}
mod b {
    pub fn foo() {
        ::a::foo(); // call `a`'s foo function
        // In Rust 2018, `::a` would be interpreted as the crate `a`.
    }
}
# fn main() {}
```

### `self`

`self` 解析相对于当前模块的路径，`self` 仅可以用作路径段开头，没有前置 `::`。

```rust
fn foo() {}
fn bar() {
    self::foo();
}
# fn main() {}
```

### `Self`

`Self`（大写“S”）用于引用 [traits] 和[实现][implementations]的类型。

`Self` 仅可以用作路径段开头，没有前置 `::`。

```rust
trait T {
    type Item;
    const C: i32;
    // `Self` will be whatever type that implements `T`.
    fn new() -> Self;
    // `Self::Item` will be the type alias in the implementation.
    fn f(&self) -> Self::Item;
}
struct S;
impl T for S {
    type Item = i32;
    const C: i32 = 9;
    fn new() -> Self {           // `Self` is the type `S`.
        S
    }
    fn f(&self) -> Self::Item {  // `Self::Item` is the type `i32`.
        Self::C                  // `Self::C` is the constant value `9`.
    }
}
```

### `super`

路径中的 `super` 解析为父模块。它仅能被用在路径的前导段，可以置于 `self` 路径段之后。

```rust
mod a {
    pub fn foo() {}
}
mod b {
    pub fn foo() {
        super::a::foo(); // call a's foo function
    }
}
# fn main() {}
```

`super` 可以在第一个 `super` 或 `self` 之后重复多次，以引用祖先模块。

```rust
mod a {
    fn foo() {}

    mod b {
        mod c {
            fn foo() {
                super::super::foo(); // call a's foo function
                self::super::super::foo(); // call a's foo function
            }
        }
    }
}
# fn main() {}
```

### `crate`

`crate` 解析相对于当前 crate 的路径。`crate` 仅能用作路径端开头，没有前置 `::`。

```rust
fn foo() {}
mod a {
    fn bar() {
        crate::foo();
    }
}
# fn main() {}
```

### `$crate`

`$crate` 仅用在[宏转换器][macro transcribers]中，且仅能用作路径段开头，没有前置 `::`。`$crate` 将扩展为从定义宏的 crate 的顶层访问 crate 各项的路径，而不用去考虑被调用宏所属的 crate。

```rust
pub fn increment(x: u32) -> u32 {
    x + 1
}

#[macro_export]
macro_rules! inc {
    ($x:expr) => ( $crate::increment($x) )
}
# fn main() { }
```

## 规范路径

定义在模块或者实现中的项具有一个 _规范路径_，该路径对应于其在 crate 中定义的位置。_规范路径_ 外所有其它指向这些项的路径都是别名。规范路径被定义为一个由其本身定义的路径段附加的 _路径前缀_。

尽管实现所定义的项有规范路径，但[实现][Implementations]和 [use 声明][use declarations]没有规范路径。块表达式中定义的项没有规范路径；在不具有规范路径的模块中定义的项也没有规范路径；在引用没有规范路径的项的实现中所定义的关联项（例如实现类型、trait 的实现、类型参数，或者类型参数上的绑定），都是没有规范路径的。

模块的路径前缀是该模块的规范路径。对于裸实现，被实现的项的规范路径使用<span class="parenthetical">尖括号（`<>`）</span> 包围。对于 [trait 实现][trait implementations]，在被实现的项的规范路径后面，首先跟随 `as`，然后再跟随 trait 的规范路径，整个规范路径都使用<span class="parenthetical">尖括号（`<>`）</span> 包围。

规范路径只有在给定的 crate 中才有意义。不同 crate 之间没有全局的命名空间；项的规范路也径仅在其 crate 中可标识。 

```rust
// Comments show the canonical path of the item.

mod a { // ::a
    pub struct Struct; // ::a::Struct

    pub trait Trait { // ::a::Trait
        fn f(&self); // a::Trait::f
    }

    impl Trait for Struct {
        fn f(&self) {} // <::a::Struct as ::a::Trait>::f
    }

    impl Struct {
        fn g(&self) {} // <::a::Struct>::g
    }
}

mod without { // ::without
    fn canonicals() { // ::without::canonicals
        struct OtherStruct; // None

        trait OtherTrait { // None
            fn g(&self); // None
        }

        impl OtherTrait for OtherStruct {
            fn g(&self) {} // None
        }

        impl OtherTrait for ::a::Struct {
            fn g(&self) {} // None
        }

        impl ::a::Trait for OtherStruct {
            fn f(&self) {} // None
        }
    }
}

# fn main() {}
```

[_GenericArgs_]: #paths-in-expressions
[_Lifetime_]: trait-bounds.md
[_Type_]: types.md#type-expressions
[item]: items.md
[variable]: variables.md
[implementations]: items/implementations.md
[use declarations]: items/use-declarations.md
[IDENTIFIER]: identifiers.md
[`use`]: items/use-declarations.md
[attributes]: attributes.md
[expressions]: expressions.md
[macro transcribers]: macros-by-example.md
[macros]: macros-by-example.md
[patterns]: patterns.md
[trait implementations]: items/implementations.md#trait-implementations
[traits]: items/traits.md
[visibility]: visibility-and-privacy.md
