# 类型参数和生存期参数

>[generics.md](https://github.com/rust-lang/reference/blob/master/src/items/generics.md)\
>commit: 7fd47ef4786f86ccdb5d6f0f198a6a9fdec5497c \
>本章译文最后维护日期：2021-1-17

> **<sup>句法</sup>**\
> _GenericParams_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `<` `>`\
> &nbsp;&nbsp;  | `<` (_GenericParam_ `,`)<sup>\*</sup> _GenericParam_ `,`<sup>?</sup> `>`
>
> _GenericParam_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> ( _LifetimeParam_ | _TypeParam_ | _ConstParam_ )
>
> _LifetimeParam_ :\
> &nbsp;&nbsp; [LIFETIME_OR_LABEL]&nbsp;( `:` [_LifetimeBounds_] )<sup>?</sup>
>
> _TypeParam_ :\
> &nbsp;&nbsp; [IDENTIFIER]( `:` [_TypeParamBounds_]<sup>?</sup> )<sup>?</sup> ( `=` [_Type_] )<sup>?</sup>
>
> _ConstParam_:\
> &nbsp;&nbsp; `const` [IDENTIFIER] `:` [_Type_]

[函数][Functions]、[类型别名][type aliases]、[结构体][structs]、[枚举][enumerations]、[联合体][unions]、[trait][traits] 和[实现][implementations]可以通过类型参数、常量参数和生存期参数达到*参数化*配置的的效果。这些参数在尖括号<span class="parenthetical">（`<...>`）</span>中列出，通常都是紧跟在程序项名称之后和程序项的定义之前。对于实现，因为它没有名称，那它们就直接位于关键字 `impl` 之后。泛型参数的申明顺序是生存期参数在最前面，然后是类型参数，最后是常量参数。

下面给出一些带类型参数、常量参数和生存期参数的程序项的示例：

```rust
fn foo<'a, T>() {}
trait A<U> {}
struct Ref<'a, T> where T: 'a { r: &'a T }
struct InnerArray<T, const N: usize>([T; N]);
```

泛型参数在声明它们的程序项定义的范围内有效。它们不是函数体中声明的程序项，这个在[程序项声明][item declarations]中有讲述。

[引用][References]、[裸指针][raw pointers]、[数组][arrays]、[切片][arrays]、[元组][tuples]和[函数指针][function pointers]也有生存期参数或类型参数，但这些程序项不能使用路径句法去引用。

### 常量泛型

*常量泛型参数*允许程序项在常量值上泛型化。const标识符为常量参数引入了一个名称，并且该程序项的所有实例必须用给定类型的值去实例化该参数。

<!-- TODO: update above to say "introduces a name in the [value namespace]" once namespaces are added. -->

常量参数类型值允许为：`u8`, `u16`, `u32`, `u64`, `u128`, `usize`, `i8`, `i16`, `i32`, `i64`, `i128`, `isize`, `char` 和 `bool` 这些类型。

常量参数可以在任何可以使用[常量项][Const item]的地方使用，但在[类型][type]或[数组定义中的重复表达式][array repeat expression]中使用时，必须如下所述是独立的。也就是说，它们可以在以下地方上允许：

1. 可以用于类型内部，用它来构成所涉及的程序项签名的一部分。
2. 作为常量表达式的一部分，用于定义[关联常量项][associated const]，或作为[关联类型][associated type]的形参。
3. 作为程序项里的任何函数体中的任何运行时表达式中的值。
4. 作为程序项中任何函数体中使用到的任何类型的参数。
5. 作为程序项中任何字段类型的一部分使用。

```rust
// 可以使用常量泛型参数的示例。

// 用于程序项本身的签名
fn foo<const N: usize>(arr: [i32; N]) {
    // 在函数体中用作类型。
    let x: [i32; N];
    // 用作表达。
    println!("{}", N * 2);
}

// 用作结构体的字段
struct Foo<const N: usize>([i32; N]);

impl<const N: usize> Foo<N> {
    // 用作关联常数
    const CONST: usize = N * 4;
}

trait Trait {
    type Output;
}

impl<const N: usize> Trait for Foo<N> {
    // 用作关联类型
    type Output = [i32; N];
}
```

```rust,compile_fail
// 不能使用常量泛型参数的示例
fn foo<const N: usize>() {
    // 能在函数体中的程序项定义中使用
    const BAD_CONST: [usize; N] = [1; N];
    static BAD_STATIC: [usize; N] = [1; N];
    fn inner(bad_arg: [usize; N]) {
        let bad_value = N * 2;
    }
    type BadAlias = [usize; N];
    struct BadStruct([usize; N]);
}
```

作为进一步的限制，常量只能作为[类型][type]或[数组定义中的重复表达式][array repeat expression]中的独立实参出现。在这种上下文限制下，它们只能以单段[路径表达式][path expression]的形式使用（例如 `N` 或以[块][block]`{N}` 的形式出现）。也就是说，它们不能与其他表达式结合使用。

```rust,compile_fail
// 不能使用常量参数的示例。

// 不允许在类型中的表达式中组合使用，例如这里的返回类型中的算术表达式
fn bad_function<const N: usize>() -> [u8; {N + 1}] {
    // 同样的，这种情况也不允许在数组定义里的重复表达式中使用
    [1; {N + 1}]
}
```

[路径][path]中的常量实参指定了该程序项使用的常量值。实参必须是常量形参所属类型的[常量表达式][const expression]。常量表达式必须是[块表达式][block]（用花括号括起来），除非它是单独路径段（一个[标识符][IDENTIFIER]）或一个[字面量][literal]（此字面量可以是以 `-` 打头的 token）。

> **注意**：这种句法限制是必要的，用以避免在解析类型内部的表达式时可能会导致无限递归(infinite lookahead)。

```rust
fn double<const N: i32>() {
    println!("doubled: {}", N * 2);
}

const SOME_CONST: i32 = 12;

fn example() {
    // 常量参数的使用示例。
    double::<9>();
    double::<-123>();
    double::<{7 + 8}>();
    double::<SOME_CONST>();
    double::<{ SOME_CONST + 5 }>();
}
```

当存在歧义时，如果泛型参数可以同时被解析为类型或常量参数，那么它总是被解析为类型。在块表达式中放置实参可以强制将其解释为常量实参。

<!-- TODO: Rewrite the paragraph above to be in terms of namespaces, once
    namespaces are introduced, and it is clear which namespace each parameter
    lives in. -->

```rust,compile_fail
type N = u32;
struct Foo<const N: usize>;
// 下面用法是错误的，因为 `N` 被解释为类型别名。
fn foo<const N: usize>() -> Foo<N> { todo!() } // ERROR
// 可以使用花括号来强制将 `N` 解释为常量形参。
fn bar<const N: usize>() -> Foo<{ N }> { todo!() } // ok
```

与类型参数和生存期参数不同，常量参数可以声明而不必在被它参数化的程序项中使用，但和[泛型实现][generic implementations]关联的实现例外：

```rust,compile_fail
// ok
struct Foo<const N: usize>;
enum Bar<const M: usize> { A, B }

// ERROR: 参数未使用
struct Baz<T>;
struct Biz<'a>;
struct Unconstrained;
impl<const N: usize> Unconstrained {}
```

当处理 trait约束时，在确定是否满足相关约束时，不会考虑常量参数的所有实现的穷尽性。例如，在下面的例子中，即使实现了 `bool`类型的所有可能的常量值，仍会报错提示 trait约束不满足。

```rust,compile_fail
struct Foo<const B: bool>;
trait Bar {}
impl Bar for Foo<true> {}
impl Bar for Foo<false> {}

fn needs_bar(_: impl Bar) {}
fn generic<const B: bool>() {
    let v = Foo::<B>;
    needs_bar(v); // ERROR: trait约束 `Foo<B>: Bar` 未被满足
}
```

## where子句

> **<sup>句法</sup>**\
> _WhereClause_ :\
> &nbsp;&nbsp; `where` ( _WhereClauseItem_ `,` )<sup>\*</sup> _WhereClauseItem_ <sup>?</sup>
>
> _WhereClauseItem_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _LifetimeWhereClauseItem_\
> &nbsp;&nbsp; | _TypeBoundWhereClauseItem_
>
> _LifetimeWhereClauseItem_ :\
> &nbsp;&nbsp; [_Lifetime_] `:` [_LifetimeBounds_]
>
> _TypeBoundWhereClauseItem_ :\
> &nbsp;&nbsp; _ForLifetimes_<sup>?</sup> [_Type_] `:` [_TypeParamBounds_]<sup>?</sup>
>
> _ForLifetimes_ :\
> &nbsp;&nbsp; `for` `<` [_GenericParams_](#generic-parameters) `>`

*where子句*提供了另一种方法来为类型参数和生存期参数指定约束(bound)，甚至可以为非类型参数的类型指定约束。

关键字`for` 可以用来引入[高阶生存期参数][higher-ranked lifetimes]。它只允许在 [_LifetimeParam_] 参数上使用。

定义程序项时，其约束没有使用程序项的泛型参数或[高阶生存期参数][higher-ranked lifetimes]，这样是可以通过编译器的安全检查，但这样的做法将必然导致错误。

在定义程序项时，编译器还会检查某些泛型参数的类型是否存在 [`Copy`]、[`Clone`] 和 [`Sized`] 这些约束。将`Copy` 或 `Clone` 作为可变引用、[trait object]或[slice][arrays]这些程序项的约束是错误的，或将 `Sized` 作为 trait对象或切片的约束也是错误的。

```rust,compile_fail
struct A<T>
where
    T: Iterator,            // 可以用 A<T: Iterator> 来替代
    T::Item: Copy,
    String: PartialEq<T>,
    i32: Default,           // 允许，但没什么用
    i32: Iterator,          // 错误: 此 trait约束不适合，i32 没有实现 Iterator
    [T]: Copy,              // 错误: 此 trait约束不适合，切片上不能有此 trait约束
{
    f: T,
}
```

## 属性

泛型生存期参数和泛型类型参数允许[属性][attributes]，但在目前这个位置还没有任何任何有意义的内置属性，但用户可能可以通过自定义的派生属性来设置一些有意义的属性。

下面示例演示如何使用自定义派生属性修改泛型参数的含义。

<!-- ignore: requires proc macro derive -->
```rust,ignore
// 假设 MyFlexibleClone 的派生项将 `my_flexible_clone` 声明为它可以理解的属性。
#[derive(MyFlexibleClone)]
struct Foo<#[my_flexible_clone(unbounded)] H> {
    a: *const H
}
```

[array repeat expression]: ../expressions/array-expr.md
[arrays]: ../types/array.md
[associated const]: associated-items.md#associated-constants
[associated type]: associated-items.md#associated-types
[block]: ../expressions/block-expr.md
[const contexts]: ../const_eval.md#const-context
[const expression]: ../const_eval.md#constant-expressions
[const item]: constant-items.md
[enumerations]: enumerations.md
[functions]: functions.md
[function pointers]: ../types/function-pointer.md
[generic implementations]: implementations.md#generic-implementations
[higher-ranked lifetimes]: ../trait-bounds.md#higher-ranked-trait-bounds
[implementations]: implementations.md
[item declarations]: ../statements.md#item-declarations
[item]: ../items.md
[literal]: ../expressions/literal-expr.md
[path]: ../paths.md
[path expression]: ../expressions/path-expr.md
[raw pointers]: ../types/pointer.md#raw-pointers-const-and-mut
[references]: ../types/pointer.md#shared-references-
[`Clone`]: ../special-types-and-traits.md#clone
[`Copy`]: ../special-types-and-traits.md#copy
[`Sized`]: ../special-types-and-traits.md#sized
[structs]: structs.md
[tuples]: ../types/tuple.md
[trait object]: ../types/trait-object.md
[traits]: traits.md
[type aliases]: type-aliases.md
[type]: ../types.md
[unions]: unions.md
[attributes]: ../attributes.md

<!-- 2021-1-20-->
<!-- checked -->
