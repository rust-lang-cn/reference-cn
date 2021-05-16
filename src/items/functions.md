# 函数

>[functions.md](https://github.com/rust-lang/reference/blob/master/src/items/functions.md)\
>commit: 245b8336818913beafa7a35a9ad59c85f28338fb \
>本章译文最后维护日期：2021-5-6

> **<sup>句法</sup>**\
> _Function_ :\
> &nbsp;&nbsp; _FunctionQualifiers_ `fn` [IDENTIFIER]&nbsp;[_GenericParams_]<sup>?</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp; `(` _FunctionParameters_<sup>?</sup> `)`\
> &nbsp;&nbsp; &nbsp;&nbsp; _FunctionReturnType_<sup>?</sup> [_WhereClause_]<sup>?</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp; ( [_BlockExpression_] | `;` )
>
> _FunctionQualifiers_ :\
> &nbsp;&nbsp; `const`<sup>?</sup> `async`[^async-edition]<sup>?</sup> `unsafe`<sup>?</sup> (`extern` _Abi_<sup>?</sup>)<sup>?</sup>
>
> _Abi_ :\
> &nbsp;&nbsp; [STRING_LITERAL] | [RAW_STRING_LITERAL]
>
> _FunctionParameters_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _SelfParam_ `,`<sup>?</sup>\
> &nbsp;&nbsp; | (_SelfParam_ `,`)<sup>?</sup> _FunctionParam_ (`,` _FunctionParam_)<sup>\*</sup> `,`<sup>?</sup>
>
> _SelfParam_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> ( _ShorthandSelf_ | _TypedSelf_ )
>
> _ShorthandSelf_ :\
> &nbsp;&nbsp;  (`&` | `&` [_Lifetime_])<sup>?</sup> `mut`<sup>?</sup> `self`
>
> _TypedSelf_ :\
> &nbsp;&nbsp; `mut`<sup>?</sup> `self` `:` [_Type_]
>
> _FunctionParam_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> (
>   _FunctionParamPattern_ | `...` | [_Type_] [^fn-param-2015]
> )
>
> _FunctionParamPattern_ :\
> &nbsp;&nbsp; [_PatternNoTopAlt_] `:` ( [_Type_] | `...` )
>
> _FunctionReturnType_ :\
> &nbsp;&nbsp; `->` [_Type_]
>
> [^async-edition]: 限定符`async`不能在 2015版中使用。
>
> [^fn-param-2015]: 在2015版中，只有类型的函数参数只允许出现在[trait项][trait item]的关联函数中。

*函数*由一个[块][block]以及一个名称和一组参数组成。除了名称，其他的都是可选的。函数使用关键字 `fn` 声明。函数可以声明一组*输入*[*变量*][variables]作为参数，调用者通过它向函数传递参数，函数完成后，它再将带有*输出*[*类型*][type]的结果值返回给调用者。

当一个*函数*被引用时，该函数会产生一个相应的零尺寸[*函数项类型(function item type)*][*function item type*]的一等(first-class)*值*，调用该值就相当于直接调用该函数。

举个简单的定义函数的例子:
```rust
fn answer_to_life_the_universe_and_everything() -> i32 {
    return 42;
}
```

## 函数参数

和 `let`绑定一样，函数参数是不可反驳型[模式][patterns]，所以任何在 let绑定中有效的模式都可以有效应用在函数参数上:

```rust
fn first((value, _): (i32, i32)) -> i32 { value }
```

如果第一个参数是一个*SelfParam*类型的参数，这表明该函数是一个[方法][method]。带 self参数的函数只能在 [trait] 或[实现][implementation]中作为[关联函数][associated function]出现。

带有 `...`token 的参数表示此函数是一个[可变参数函数][variadic function]，`...` 只能作为[外部块][external block]里的函数的最后一个参数使用。可变参数可以有一个可选标识符，例如 `args: ...`。

## 函数体

函数的块在概念上被包装进在一个块中，该块绑定该函数的参数模式，然后返回(`return`)该函数的块的值。这意味着如果轮到块的*尾部表达式(tail expression)*被求值计算了，该块将结束，求得的值将被返回给调用者。通常，程序执行时如果流碰到函数体中的显式返回表达式(return expression)，就会截断那个隐式的最终表达式的执行。

例如，上面例子里函数的行为就像下面被改写的这样:

<!-- ignore: example expansion -->
```rust,ignore
// argument_0 是调用者真正传过来的第一个参数
let (value, _) = argument_0;
return {
    value
};
```

没有主体块的函数以分号结束。这种形式只能出现在 [trait] 或[外部块][external block]中。

## 泛型函数

*泛型函数*允许在其签名中出现一个或多个*参数化类型(parameterized types)*。每个类型参数必须在函数名后面的尖括号封闭逗号分隔的列表中显式声明。

```rust
// foo 是建立在 A 和 B 基础上的泛型函数

fn foo<A, B>(x: A, y: B) {
# }
```

在函数签名上和函数体内部，类型参数的名称可以被用作类型名。可以为类型参数指定 [trait][Trait]约束，以允许对该类型的值调用这些 trait 的方法。这种约束是使用 `where`句法指定的：

```rust
# use std::fmt::Debug;
fn foo<T>(x: T) where T: Debug {
# }
```

当使用泛型函数时，它的（类型参数的）类型会基于调用的上下文被实例化。例如，这里调用 `foo` 函数:

```rust
use std::fmt::Debug;

fn foo<T>(x: &[T]) where T: Debug {
    // 省略细节
}

foo(&[1, 2]);
```

将用 `i32` 实例化类型参数 `T`。

类型参数也可以在函数名之后的末段[路径][path]组件(trailing path component，即 `::<type>`)中显式地提供。如果没有足够的上下文来确定类型参数，那么这可能是必要的。例如：`mem::size_of::<u32>() == 4`。

## 外部函数限定符

外部函数限定符(`extern`)可以提供那些能通过特定 ABI 才能调用的函数的*定义*：

<!-- ignore: fake ABI -->
```rust,ignore
extern "ABI" fn foo() { /* ... */ }
```

这些通常与提供了函数*声明*的[外部块][external block]程序项一起使用，这样就可以调用此函数而不必同时提供此函数的*定义*：

<!-- ignore: fake ABI -->
```rust,ignore
extern "ABI" {
  fn foo(); /* no body */
}
unsafe { foo() }
```

当函数的 `FunctionQualifiers` 句法规则中的 `"extern" Abi?*` 选项被省略时，会默认使用 `"Rust"` 类型的 ABI。例如:

```rust
fn foo() {}
```

等价于：

```rust
extern "Rust" fn foo() {}
```

使用 `"Rust"` 之外的 ABI 可以让 Rust 中声明的函数被其他编程语言调用。比如下面声明了一个可以从 C 中调用的函数：

```rust
// 使用 "C" ABI 声明一个函数
extern "C" fn new_i32() -> i32 { 0 }

// 使用 "stdcall" ABI声明一个函数
# #[cfg(target_arch = "x86_64")]
extern "stdcall" fn new_i32_stdcall() -> i32 { 0 }
```

与[外部块][external block]一样，当使用关键字 `extern` 而省略 `"ABI` 时，ABI 默认使用的是 `"C"`。也就是说这个：

```rust
extern fn new_i32() -> i32 { 0 }
let fptr: extern fn() -> i32 = new_i32;
```

等价于：

```rust
extern "C" fn new_i32() -> i32 { 0 }
let fptr: extern "C" fn() -> i32 = new_i32;
```

非 `"Rust"` 的 ABI 函数不支持与 Rust 函数完全相同的展开(unwind)方式。因此展开进程过这类 ABI 函数的尾部时就会导致该进程被终止。（译者注：展开是逆向的。）

> **注意**: `rustc` 背后的 LLVM 是通过执行一个非法指令来实现中止进程的功能的。

## 常量函数

使用关键字 `const` 限定的函数是[常量(const)函数][const functions]，[元组结构体][tuple struct]构造函数和[元组变体][tuple variant]构造函数也是如此。可以在[常量上下文][const context]中调用*常量函数*。

常量函数不允许是 [async](#async-functions)类型的，并且不能使用 [`extern`函数限定符](#extern-function-qualifier)。

## 异步函数

函数可以被限定为异步的，这还可以与 `unsafe` 限定符结合在一起使用：

```rust,edition2018
async fn regular_example() { }
async unsafe fn unsafe_example() { }
```

异步函数在调用时不起作用：相反，它们将参数捕获进一个 future。当该函数被轮询(polled)时，该 future 将执行该函数的函数体。

一个异步函数大致相当于返回一个以 [`async move`块][async-blocks]为代码体的 [`impl Future`] 的函数：

```rust,edition2018
// 源代码
async fn example(x: &str) -> usize {
    x.len()
}
```

大致等价于：

```rust,edition2018
# use std::future::Future;
// 脱糖后的
fn example<'a>(x: &'a str) -> impl Future<Output = usize> + 'a {
    async move { x.len() }
}
```

实际的脱糖(desugaring)过程相当复杂：

- 脱糖过程中的返回类型被假定捕获了 *`async fn`声明*中的所有的生存期参数（包括省略的）。这可以在上面的脱糖示例中看到：（我们）显式地给它补上了一个生存期参数 `'a`，因此捕捉到 `'a`。
- 代码体中的 [`async move`块][async blocks]捕获所有函数参数，包括未使用或绑定到 `_` 模式的参数。参数销毁方面，除了销毁动作需要在返回的 future 完全执行(fully awaited)完成后才会发生外，可以确保异步函数参数的销毁顺序与函数非异步时的顺序相同。

有关异步效果的详细信息，请参见 [`async`块][async-blocks]。

[async-blocks]: ../expressions/block-expr.md#async-blocks
[`impl Future`]: ../types/impl-trait.md

> **版本差异**: 异步函数只能从 Rust 2018 版才开始可用。

### `async` 和 `unsafe` 的联合使用

声明一个既异步又非安全(`unsafe`)的函数是合法的。调用这样的函数是非安全的，并且（像任何异步函数一样）会返回一个 future。这个 future 只是一个普通的 future，因此“await”它不需要一个 `unsafe` 的上下文：

```rust,edition2018
// 等待这个返回的 future 相当于解引用 `x`。
//
// 安全条件: 在返回的 future 执行完成前，`x` 必须能被安全解引用
async unsafe fn unsafe_example(x: *const i32) -> i32 {
  *x
}

async fn safe_example() {
    // 起初调用上面这个函数时需要一个非安全(`unsafe`)块：
    let p = 22;
    let future = unsafe { unsafe_example(&p) };

    // 但是这里非安全(`unsafe`)块就没必要了，这里能正常读到 `p`:
    let q = future.await;
}
```

请注意，此行为是对返回 `impl Future` 的函数进行脱糖处理的结果——在本例中，我们脱糖处理生成的函数是一个非安全(`unsafe`)函数，这个非安全(`unsafe`)函数返回值与原始定义的函数的返回值仍保持一致。

非安全限定在异步函数上的使用方式与它在其他函数上的使用方式完全相同：只是它表示该函数会要求它的调用者提供一些额外的义务/条件来确保该函数的健全性(soundness)。与任何其他非安全函数一样，这些条件可能会超出初始调用本身——例如，在上面的代码片段中， 函数 `unsafe_example` 将指针 `x` 作为参数，然后（在执行 await 时）解引用了对该指针的引用。这意味着在 future 完成执行之前，`x` 必须是有效的，调用者有责任确保这一点。

## 函数上的属性

在函数上允许使用[外部属性][attributes]，也允许在[函数体][block]中的 `{` 后面直接放置[内部属性][attributes]。

下面这个例子显示了一个函数的内部属性。该函数的文档中只会出现单词“Example”。

```rust
fn documented() {
    #![doc = "Example"]
}
```

> 注意：除了 lint检查类属性，函数上一般惯用的还是外部属性。

在函数上有意义的属性是 [`cfg`]、[`cfg_attr`]、[`deprecated`]、[`doc`]、[`export_name`]、[`link_section`]、[`no_mangle`]、[lint检查类属性][the lint check attributes]、[`must_use`]、[过程宏属性][the procedural macro attributes]、[测试类属性][the testing attributes]和[优化提示类属性][the optimization hint attributes]。函数也可以接受属性宏。

## 函数参数上的属性

函数参数允许使用[外部属性][attributes]，允许的[内置属性][built-in attributes]仅限于 `cfg`、`cfg_attr`、`allow`、`warn`、`deny` 和 `forbid`。

```rust
fn len(
    #[cfg(windows)] slice: &[u16],
    #[cfg(not(windows))] slice: &[u8],
) -> usize {
    slice.len()
}
```

应用于程序项的过程宏属性所使用的惰性辅助属性也是允许的，但是要注意不要在最终（输出）的 `TokenStream` 中包含这些惰性属性。

例如，下面的代码定义了一个未在任何地方正式定义的惰性属性 `some_inert_attribute`，而 `some_proc_macro_attribute` 过程宏负责检测它的存在，并从输出 token流 `TokenStream` 中删除它。

<!-- ignore: requires proc macro -->
```rust,ignore
#[some_proc_macro_attribute]
fn foo_oof(#[some_inert_attribute] arg: u8) {
}
```

[IDENTIFIER]: ../identifiers.md
[RAW_STRING_LITERAL]: ../tokens.md#raw-string-literals
[STRING_LITERAL]: ../tokens.md#string-literals
[_BlockExpression_]: ../expressions/block-expr.md
[_GenericParams_]: generics.md
[_Lifetime_]: ../trait-bounds.md
[_PatternNoTopAlt_]: ../patterns.md
[_Type_]: ../types.md#type-expressions
[_WhereClause_]: generics.md#where-clauses
[_OuterAttribute_]: ../attributes.md
[const context]: ../const_eval.md#const-context
[const functions]: ../const_eval.md#const-functions
[tuple struct]: structs.md
[tuple variant]: enumerations.md
[external block]: external-blocks.md
[path]: ../paths.md
[block]: ../expressions/block-expr.md
[variables]: ../variables.md
[type]: ../types.md#type-expressions
[*function item type*]: ../types/function-item.md
[Trait]: traits.md
[attributes]: ../attributes.md
[`cfg`]: ../conditional-compilation.md#the-cfg-attribute
[`cfg_attr`]: ../conditional-compilation.md#the-cfg_attr-attribute
[the lint check attributes]: ../attributes/diagnostics.md#lint-check-attributes
[the procedural macro attributes]: ../procedural-macros.md
[the testing attributes]: ../attributes/testing.md
[the optimization hint attributes]: ../attributes/codegen.md#optimization-hints
[`deprecated`]: ../attributes/diagnostics.md#the-deprecated-attribute
[`doc`]: https://doc.rust-lang.org/rustdoc/the-doc-attribute.html
[`must_use`]: ../attributes/diagnostics.md#the-must_use-attribute
[patterns]: ../patterns.md
[`export_name`]: ../abi.md#the-export_name-attribute
[`link_section`]: ../abi.md#the-link_section-attribute
[`no_mangle`]: ../abi.md#the-no_mangle-attribute
[built-in attributes]: ../attributes.html#built-in-attributes-index
[trait item]: traits.md
[method]: associated-items.md#methods
[associated function]: associated-items.md#associated-functions-and-methods
[implementation]: implementations.md
[variadic function]: external-blocks.md#variadic-functions

<!-- 2021-1-17-->
<!-- checked -->
