## 过程宏

*过程宏* 允许在执行函数时创建语法扩展。过程宏有三种类型：

* [类函数宏][Function-like macros] - `custom!(...)`
* [派生宏][Derive macros] - `#[derive(CustomDerive)]`
* [属性宏][Attribute macros] - `#[CustomAttribute]`

过程宏允许您在编译时运行对 Rust 语法进行操作的代码，同时使用并生成 Rust 语法。您可以将过程宏看作是从一个 AST 到另一个 AST 的函数。

过程宏必须在 crate 中定义，其 [crate 类型][crate type]为 `proc-macro`。

> **注**：使用 Cargo 时，过程宏 crates 使用清单中的 `proc-macro` 建定义：
>
> ```toml
> [lib]
> proc-macro = true
> ```

作为函数，它们必须要么返回语法、panic，要么无休止地循环。返回的语法根据过程宏的类型替换或添加语法。编译器捕获 panic 并将其转化为编译器错误。编译器不会捕获无休止地循环，而是被挂起。

过程宏在编译期间运行，因此具有与编译器相同的资源。例如，过程宏与编译器可以访问的标准输入、错误和输出是相同的。类似地，文件访问也是一样的。因此，过程宏与 [Cargo 构建脚本][Cargo's
build scripts]具有相同的安全问题。

过程宏有两种报告错误的方法。首先是 panic，第二种是发起一个[编译错误][`compile_error`]宏调用。

### `proc_macro` crate

过程宏 crates 几乎总是链接到编译器提供的 [`proc_macro` crate]。`proc_macro` crate 提供编写过程宏所需的类型和工具，以使其更容易。

`proc_macro` crate 主要包含 [`TokenStream`] 类型。过程宏在 *记号流* 上操作，而不是在 AST 节点上操作，对于编译器和过程宏来说，这是一个随着时间推移更加稳定的接口。*记号流* 大致相当于 `Vec<TokenTree>`，其中 `TokenTree` 可以粗略地看作是词法标记。例如，`foo` 是一个 `Ident` 标记，`.` 是一个 `Punct` 标记，而 `1.2` 是一个`字面量`标记。与 `Vec<TokenTree>` 不同，`TokenStream` 类型的克隆成本较低。

所有的标记都有一个关联的 `Span`。`Span` 是一个不透明的值，可以生成但不能修改。`Span` 表示程序中源代码的作用域，主要用于错误报告。您可以修改任何标记的 `Span`。

### 卫生（hygiene）过程宏

过程宏是 *不卫生的*。这意味着它们表现地好像输出记号流是直接内联写入代码一样。这意味着它受到外部项的影响，也会影响外部对它的导入。

鉴于此限制，宏编写者需要小心，以确保他们编写的宏在尽可能多的上下文中工作。这通常包括对库中的项使用绝对路径（例如，`::std::option::Option` 而不是 `Option`），或者确保生成的函数具有不太可能与其他函数冲突的名称（比如，`__internal_foo` 而不是 `foo`）。

### 类函数过程宏

*类函数过程宏* 是使用宏调用运算符（`!`）调用的过程宏。

这些宏由带有 `proc_macro` [属性][attribute]和 `(TokenStream) -> TokenStream` 签名的[公有][public]可见性[函数][function]定义。输入 [`TokenStream`] 是宏调用的分隔符内的内容，输出 [`TokenStream`] 替换整个宏调用。

例如，以下宏定义忽略其输入，并将函数`返回值`输出到其作用域中。

<!-- ignore: test doesn't support proc-macro -->
```rust,ignore
# #![crate_type = "proc-macro"]
extern crate proc_macro;
use proc_macro::TokenStream;

#[proc_macro]
pub fn make_answer(_item: TokenStream) -> TokenStream {
    "fn answer() -> u32 { 42 }".parse().unwrap()
}
```

然后，我们用它来打印 “42” 到标准输出。

<!-- ignore: requires external crates -->
```rust,ignore
extern crate proc_macro_examples;
use proc_macro_examples::make_answer;

make_answer!();

fn main() {
    println!("{}", answer());
}
```

类函数过程宏可以在任何宏调用位置被调用，宏调用位置包括：[语句][statements]、[表达式][expressions]、[模式][patterns]、[类型表达式][type
expressions]、[项][item]位置（包括[`外部`块][`extern` blocks]中的项）、内置和 trait [实现][implementations]，以及 [trait 定义][trait definitions]。

### 派生宏

*派生宏* 定义[`派生`属性][`derive` attribute]的新输入。这些宏可以在给定[结构体][struct]、[枚举][enum]，或[联合][union]的记号流的情况下创建新[项][items]。它们还可以定义[派生宏助手属性][derive macro helper attributes]。

自定义派生宏由带有 `proc_macro_derive` 属性和 `(TokenStream) -> TokenStream` 签名的[公有][public]可见性[函数][function]定义。

输入 [`TokenStream`] 是具有`派生`属性的项的记号流。输出 [`TokenStream`] 必须是一组项，然后附加到来自输入
[`TokenStream`] 的项所在的[模块][module]或[块][block]。

下面是派生宏的一个示例。它没有对输入执行任何有用的操作，只是附加一个函数`返回值`。

<!-- ignore: test doesn't support proc-macro -->
```rust,ignore
# #![crate_type = "proc-macro"]
extern crate proc_macro;
use proc_macro::TokenStream;

#[proc_macro_derive(AnswerFn)]
pub fn derive_answer_fn(_item: TokenStream) -> TokenStream {
    "fn answer() -> u32 { 42 }".parse().unwrap()
}
```

然后使用上述派生宏：

<!-- ignore: requires external crates -->
```rust,ignore
extern crate proc_macro_examples;
use proc_macro_examples::AnswerFn;

#[derive(AnswerFn)]
struct Struct;

fn main() {
    assert_eq!(42, answer());
}
```

#### 派生宏助手属性

派生宏可以将附加[属性][attributes]添加到它们所在[项][item]的作用域中。所述附加属性被称为 *派生宏助手属性*。这些属性是[惰性的][inert]，它们的唯一目的是注入到定义它们的派生宏中。也就是说，所有宏都可以看到它们。

定义助手属性的方法是在
`proc_macro_derive` 宏中放入一个`属性`键，该宏带有一个逗号分隔的标识符列表，这些标识符是助手属性的名称。

例如，下面的派生宏定义了一个助手属性
`helper`，但根本不会对其执行任何操作。

<!-- ignore: test doesn't support proc-macro -->
```rust,ignore
# #![crate_type="proc-macro"]
# extern crate proc_macro;
# use proc_macro::TokenStream;

#[proc_macro_derive(HelperAttr, attributes(helper))]
pub fn derive_helper_attr(_item: TokenStream) -> TokenStream {
    TokenStream::new()
}
```

然后在结构体的派生宏中使用：

<!-- ignore: requires external crates -->
```rust,ignore
#[derive(HelperAttr)]
struct Struct {
    #[helper] field: ()
}
```

### 属性宏

*属性宏* 定义可以附加到[项][items]的新[外部属性][attributes]，包括[`外部`块][`extern` blocks]中的项、内置和 trait
[实现][implementations]，以及 [trait 定义][trait definitions]。

属性宏由带有
`proc_macro_attribute` [属性][attribute]的[公有][public]可见性[函数][function]定义，该属性的签名为 `(TokenStream,
TokenStream) -> TokenStream`。第一个 [`TokenStream`] 是属性名称后面带分隔符的标记树，不包括外部分隔符。如果该属性作为裸属性名写入，则属性 [`TokenStream`] 为空。第二个 [`TokenStream`]
是[项][item]的其余部分，包括该[项][item]的其他属性。返回的 [`TokenStream`] 将[项][item]替换为任意数量的[项][items]。

例如，这个属性宏接受输入流并按原样返回，实际上对属性并无操作。

<!-- ignore: test doesn't support proc-macro -->
```rust,ignore
# #![crate_type = "proc-macro"]
# extern crate proc_macro;
# use proc_macro::TokenStream;

#[proc_macro_attribute]
pub fn return_as_is(_attr: TokenStream, item: TokenStream) -> TokenStream {
    item
}
```

下面的示例显示了属性宏看到的字符串化[记号流][`TokenStream`s]。输出将显示在编译器的输出中。输出显示在以 “out:” 为前缀的函数后面的注释中。

<!-- ignore: test doesn't support proc-macro -->
```rust,ignore
// my-macro/src/lib.rs
# extern crate proc_macro;
# use proc_macro::TokenStream;

#[proc_macro_attribute]
pub fn show_streams(attr: TokenStream, item: TokenStream) -> TokenStream {
    println!("attr: \"{}\"", attr.to_string());
    println!("item: \"{}\"", item.to_string());
    item
}
```

<!-- ignore: requires external crates -->
```rust,ignore
// src/lib.rs
extern crate my_macro;

use my_macro::show_streams;

// Example: Basic function
#[show_streams]
fn invoke1() {}
// out: attr: ""
// out: item: "fn invoke1() { }"

// Example: Attribute with input
#[show_streams(bar)]
fn invoke2() {}
// out: attr: "bar"
// out: item: "fn invoke2() {}"

// Example: Multiple tokens in the input
#[show_streams(multiple => tokens)]
fn invoke3() {}
// out: attr: "multiple => tokens"
// out: item: "fn invoke3() {}"

// Example:
#[show_streams { delimiters }]
fn invoke4() {}
// out: attr: "delimiters"
// out: item: "fn invoke4() {}"
```

[Attribute macros]: #属性宏
[Cargo's build scripts]: ../cargo/reference/build-scripts.html
[Derive macros]: #派生宏
[Function-like macros]: #类函数过程宏
[`TokenStream`]: ../proc_macro/struct.TokenStream.html
[`TokenStream`s]: ../proc_macro/struct.TokenStream.html
[`compile_error`]: ../std/macro.compile_error.html
[`derive` attribute]: attributes/derive.md
[`extern` blocks]: items/external-blocks.md
[`macro_rules`]: macros-by-example.md
[`proc_macro` crate]: ../proc_macro/index.html
[attribute]: attributes.md
[attributes]: attributes.md
[block]: expressions/block-expr.md
[crate type]: linkage.md
[derive macro helper attributes]: #派生宏助手属性
[enum]: items/enumerations.md
[expressions]: expressions.md
[function]: items/functions.md
[implementations]: items/implementations.md
[inert]: attributes.md#active-and-inert-attributes
[item]: items.md
[items]: items.md
[module]: items/modules.md
[patterns]: patterns.md
[public]: visibility-and-privacy.md
[statements]: statements.md
[struct]: items/structs.md
[trait definitions]: items/traits.md
[type expressions]: types.md#type-expressions
[type]: types.md
[union]: items/unions.md
