# 路径

>[paths.md](https://github.com/rust-lang/reference/blob/master/src/paths.md)\
>commit: 80241b46c68380735f05fb53bd99632b87ac2872 \
>本章译文最后维护日期：2021-3-13

*路径*是一个或多个由命名空间<span class="parenthetical">限定符(`::`)</span>*逻辑*分隔的路径段(path segments)组成的序列（译者注：如果只有一个段的话，`::` 不是必须的）。如果路径仅由一个路径段组成，则它引用局部控制域(control scope)内的[程序项][item]或[变量][variable]。如果路径包含多个路径段，则总是引用程序项。

仅由标识符段组成的简单路径(simple paths)的两个示例：
<!-- ignore: syntax fragment -->
```rust,ignore
x;
x::y::z;net
```

## 路径分类

### Simple Paths
### 简单路径

> **<sup>句法</sup>**\
> _SimplePath_ :\
> &nbsp;&nbsp; `::`<sup>?</sup> _SimplePathSegment_ (`::` _SimplePathSegment_)<sup>\*</sup>
>
> _SimplePathSegment_ :\
> &nbsp;&nbsp; [IDENTIFIER] | `super` | `self` | `crate` | `$crate`

简单路径可用于[可见性][visibility]标记、[属性][attributes]、[宏][macros]和 [`use`]程序项中。示例：

```rustnet
use std::io::{self, Write};
mod m {
    #[clippy::cyclomatic_complexity = "0"]
    pub (in super) fn f1() {}
}
```

### 表达式中的路径

> **<sup>句法</sup>**\
> _PathInExpression_ :\net
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
> &nbsp;&nbsp; | `<` ( _GenericArg_ `,` )<sup>\*</sup> _GenericArg_ `,`<sup>?</sup> `>`
>
> _GenericArg_ :\net
> &nbsp;&nbsp; [_Lifetime_] | [_Type_] | _GenericArgsConst_ | _GenericArgsBinding_
>
> _GenericArgsConst_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_BlockExpression_]\
> &nbsp;&nbsp; | [_LiteralExpression_]\
> &nbsp;&nbsp; | `-` [_LiteralExpression_]\
> &nbsp;&nbsp; | [_SimplePathSegment_]
>
> _GenericArgsBinding_ :\
> &nbsp;&nbsp; [IDENTIFIER] `=` [_Type_]

表达式中的路径允许指定带有泛型参数的路径。它们在各种[表达式][expressions]和[模式][patterns]中都有使用。

token `::` 必须在泛型参数的左尖括号（`<`）的前面，以避免和小于号操作符产生混淆。这就是俗称的“涡轮鱼(turbofish)”句法。

```rust
(0..10).collect::<Vec<_>>();net
Vec::<u8>::with_capacity(1024);
```

泛型参数的顺序被限制为生存期参数，然后是类型参数，然后是常量参数，再后是相应的约束。

常量实参必须用花括号括起来，除非它们是[字面量][literal]或单段路径。

## 限定性路径

> **<sup>句法</sup>**\
> _QualifiedPathInExpression_ :\
> &nbsp;&nbsp; _QualifiedPathType_ (`::` _PathExprSegment_)<sup>+</sup>
>
> _QualifiedPathType_ :\
> &nbsp;&nbsp; `<` [_Type_] (`as` _TypePath_)? `>`
>
> _QualifiedPathInType_ :\
> &nbsp;&nbsp; _QualifiedPathType_ (`::` _TypePathSegment_)<sup>+</sup>

完全限定性路径可以用来为 [trait实现(trait implementations)][trait implementations]消除路径歧义，也可以用来指定[规范路径](#canonical-paths)。用在指定具体类型时，它支持使用如下所示的类型句法：

```rust
struct S;
impl S {
    fn f() { println!("S"); }
}
trait T1 {
    fn f() { println!("T1 f"); }
}
impl T1 for S {}
trait T2 {net
    fn f() { println!("T2 f"); }
}net
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

类型路径用于类型定义、trait约束(trait bound)、类型参数约束，以及限定性路径。

尽管在泛型参数之前允许使用 token `::`，但它不是必需的，因为在类型路径中不存在像 _PathInExpression_ 句法中那样的歧义。

```rust
# mod ops {
#     pub struct Range<T> {f1: T}
#     pub trait Index<T> {}
#     pub struct Example<'a> {f1: &'a i32}
# }
# struct S;
impl ops::Index<ops::Range<usize>> for S { /*...*/ }
fn i<'a>() -> impl Iterator<Item = ops::Example<'a>> {
    // ...net
#    const EXAMPLE: Vec<ops::Example<'static>> = Vec::new();
#    EXAMPLE.into_iter()
}
type G = std::boxed::Box<dyn std::ops::FnOnce(isize) -> isize>;
```

## 路径限定符

路径可以被各种能改变其解析方式的前导限定符限定。

### `::`

以 `::` 开头的路径被认为是全局路径，其中的路径首段在不同的版本中的解析方式有所不同。但路径中的每个标识符都必须解析为一个程序项。

> **版本差异**: 在2015版中，标识符解析从“create 根模块(crate root)”（2018版中表示为 `crate::`）开始，“create 根模块(crate root)”中包含了一系列不同的程序项，包括外部crate、默认create（如 `std` 或 `core`），以及 crate下的各种顶层程序项（包括 `use`导入）。
> 
>从 2018 版开始，以 `::` 开头的路径被解析为[外部预导入包][extern prelude]中的一个 crate。也就是说，其后必须跟一个 crate的名称。

```rust
mod a {
    pub fn foo() {}
}
mod b {
    pub fn foo() {
        ::a::foo(); // 调用 `a`'的 foo 函数
        // 在 Rust 2018 中, `::a` 会被解析为 crate `a`.
    }
}
# fn main() {}
```

### `self`

`self` 表示路径是相对于当前模块的路径。`self` 仅可以用作路径的首段，不能有前置 `::`。

```rust
fn foo() {}
fn bar() {
    self::foo();
}
# fn main() {}
```

### `Self`

`Self`（首字母大写）用于指代 [trait][traits] 和[实现][implementations]中的类型。

`Self` 仅可以用作路径的首段，不能有前置 `::`。

```rust
trait T {
    type Item;
    const C: i32;
    // `Self` 将是实现 `T` 的任何类型。
    fn new() -> Self;
    // `Self::Item` 将是实现中指定的类型的别名。
    fn f(&self) -> Self::Item;
}
struct S;
impl T for S {
    type Item = i32;
    const C: i32 = 9;
    fn new() -> Self {           // `Self` 是类型 `S`.
        S
    }
    fn f(&self) -> Self::Item {  // `Self::Item` 是类型 `i32`.
        Self::C                  // `Self::C` 是常量值 `9`.
    }
}
```

### `super`

`super` 在路径中被解析为父模块。它只能用于路径的前导段，可以置于 `self` 路径段之后。

```rust
mod a {
    pub fn foo() {}
}
mod b {
    pub fn foo() {
        super::a::foo(); // 调用 a'的 foo 函数
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
                super::super::foo(); // 调用 a'的 foo 函数
                self::super::super::foo(); // 调用 a'的 foo 函数
            }
        }
    }
}
# fn main() {}
```

### `crate`

`crate` 解析相对于当前 crate 的路径。crate 仅能用作路径的首段，不能有前置 `::`。

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

$crate 仅用在[宏转码器(macro transcriber)][macro transcribers]中，且仅能用作路径首段，不能有前置 `::`。`$crate` 将被扩展为从定义宏的 crate 的顶层访问该 crate 中的各程序项的路径，而不用去考虑宏调用发生时所在的现场 crate。

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

定义在模块或者实现中的程序项都有一个*规范路径*，该路径对应于其在其 crate 中定义的位置。在该路径之外，所有指向这些程序项的路径都是别名（路径）。规范路径被定义为一个*路径前缀*后跟一个代表程序项本身的那段路径段。

尽管实现所定义/实现的程序项有规范路径，但[实现][Implementations]作为一个整体，自身没有规范路径。[use声明][use declarations]也没有规范路径。块表达式中定义的程序项也没有规范路径。在没有规范路径的模块中定义的程序项也没有规范路径。在实现中定义的关联项(associated items)如果它指向没有规范路径的程序项——例如，实现类型(implementing type)、被实现的 trait、类型参数或类型参数上的约束——那它也没有规范路径。

模块的路径前缀就是该模块的规范路径。对于裸实现(bare implementations)来说，它里面的程序项的路径前缀是：它当前实现的程序项的规范路径，再用<span class="parenthetical">尖括号（`<>`）</span>括把此路径括起来。对于 [trait实现][trait implementations]来说，它内部的程序项的路径前缀是：*当前实现的程序项的规范路径后跟一个 `as`，再后跟 trait 本身的规范路径，然后整个使用<span class="parenthetical">尖括号（`<>`）</span>括起来。*

规范路径只在给定的 crate 中有意义。在 crate 之间没有全局命名空间，所以程序项的规范路径只在其 crate 中可标识。

```rust
// 注释解释了程序项的规范路径

mod a { // ::a
    pub struct Struct; // ::a::Struct

    pub trait Trait { // ::a::Trait
        fn f(&self); // ::a::Trait::f
    }

    impl Trait for Struct {
        fn f(&self) {} // <::a::Struct as ::a::Trait>::f
    }

    impl Struct {  // 译者注：这是一个裸实现
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

[_BlockExpression_]: expressions/block-expr.md
[_Expression_]: expressions.md
[_GenericArgs_]: #paths-in-expressions
[_Lifetime_]: trait-bounds.md
[_LiteralExpression_]: expressions/literal-expr.md
[_SimplePathSegment_]: #simple-paths
[_Type_]: types.md#type-expressions
[literal]: expressions/literal-expr.md
[item]: items.md
[variable]: variables.md
[implementations]: items/implementations.md
[use declarations]: items/use-declarations.md
[IDENTIFIER]: identifiers.md
[`use`]: items/use-declarations.md
[attributes]: attributes.md
[expressions]: expressions.md
[extern prelude]: names/preludes.md#extern-prelude
[macro transcribers]: macros-by-example.md
[macros]: macros-by-example.md
[patterns]: patterns.md
[trait implementations]: items/implementations.md#trait-implementations
[traits]: items/traits.md
[visibility]: visibility-and-privacy.md
