# 声明宏

>[macros-by-example.md](https://github.com/rust-lang/reference/blob/master/src/macros-by-example.md)\
>commit: d23f9da8469617e6c81121d9fd123443df70595d \
>本章译文最后维护日期：2021-5-6

> **<sup>句法</sup>**\
> _MacroRulesDefinition_ :\
> &nbsp;&nbsp; `macro_rules` `!` [IDENTIFIER] _MacroRulesDef_
>
> _MacroRulesDef_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `(` _MacroRules_ `)` `;`\
> &nbsp;&nbsp; | `[` _MacroRules_ `]` `;`\
> &nbsp;&nbsp; | `{` _MacroRules_ `}`
>
> _MacroRules_ :\
> &nbsp;&nbsp; _MacroRule_ ( `;` _MacroRule_ )<sup>\*</sup> `;`<sup>?</sup>
>
> _MacroRule_ :\
> &nbsp;&nbsp; _MacroMatcher_ `=>` _MacroTranscriber_
>
> _MacroMatcher_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `(` _MacroMatch_<sup>\*</sup> `)`\
> &nbsp;&nbsp; | `[` _MacroMatch_<sup>\*</sup> `]`\
> &nbsp;&nbsp; | `{` _MacroMatch_<sup>\*</sup> `}`
>
> _MacroMatch_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_Token_]<sub>_排除 $ 和 定界符_</sub>\
> &nbsp;&nbsp; | _MacroMatcher_\
> &nbsp;&nbsp; | `$` [IDENTIFIER] `:` _MacroFragSpec_\
> &nbsp;&nbsp; | `$` `(` _MacroMatch_<sup>+</sup> `)` _MacroRepSep_<sup>?</sup> _MacroRepOp_
>
> _MacroFragSpec_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `block` | `expr` | `ident` | `item` | `lifetime` | `literal`\
> &nbsp;&nbsp; | `meta` | `pat` | `path` | `stmt` | `tt` | `ty` | `vis`
>
> _MacroRepSep_ :\
> &nbsp;&nbsp; [_Token_]<sub>_排除 定界符 和 重复操作符_</sub>
>
> _MacroRepOp_ :\
> &nbsp;&nbsp; `*` | `+` | `?`
>
> _MacroTranscriber_ :\
> &nbsp;&nbsp; [_DelimTokenTree_]

`macro_rules` 允许用户以声明性的(declarative)方式定义句法扩展。我们称这种扩展形式为“声明宏（macros by example）”或简称“宏”。

每个声明宏都有一个名称和一条或多条*规则*。每条规则都有两部分：一个*匹配器(matcher)*，描述它匹配的句法；一个*转码器(transcriber)*，描述成功匹配后将执行的替代调用句法。匹配器和转码器都必须由定界符(delimiter)包围。宏可以扩展为表达式、语句、程序项（包括 trait、impl 和外来程序项）、类型或模式。

## 转码

当宏被调用时，宏扩展器(macro expander)按名称查找宏调用，并依次尝试此宏中的每条宏规则。宏会根据第一个成功的匹配进行转码；如果当前转码结果导致错误，不会再尝试进行后续匹配。在匹配时，不会执行预判；如果编译器不能明确地确定如何一个 token 一个 token 地解析宏调用，则会报错。在下面的示例中，编译器不会越过标识符，去提前查看后跟的 token 是 `)`，尽管这能帮助它明确地解析调用：

```rust,compile_fail
macro_rules! ambiguity {
    ($($i:ident)* $j:ident) => { };
}

ambiguity!(error); // 错误: 局部歧义(local ambiguity)
```

在匹配器和转码器中，token `$` 用于从宏引擎中调用特殊行为（下文[元变量][Metavariables]和[重复元][Repetitions]中有详述）。不属于此类调用的 token 将按字面意义进行匹配和转码，除了一个例外。这个例外是匹配器的外层定界符将匹配任何一对定界符。因此，比如匹配器 `(())` 将匹配 `{()}`，而 `{{}}` 不行。字符 `$` 不能按字面意义匹配或转码。

当将当前匹配的匹配段转发给另一个声明宏时，第二个宏中的匹配器看到的将是此匹配段类型的不透明抽象句法树(opaque AST)。第二个宏不能使用字面量token 来匹配匹配器中的这个匹配段，唯一可看到/使用的就是此匹配段类型一样的匹配段选择器(fragment specifier)。但匹配段类型 `ident`、`lifetime`、和 `tt` 是几个例外，它们*可以*通过字面量token 进行匹配。下面示例展示了这一限制：（译者注：匹配段选择器和匹配段，以及宏中各部件的定义可以凑合着看看译者未能翻译完成的[宏定义规范](macro-ambiguity.md)）

```rust,compile_fail
macro_rules! foo {
    ($l:expr) => { bar!($l); }
// ERROR:               ^^ no rules expected this token in macro call
}

macro_rules! bar {
    (3) => {}
}

foo!(3);
```

以下示例展示了 `tt` 类型的匹配段在成功匹配（转码）一次之后生成的 tokens 如何能够再次直接匹配：

```rust
// 成功编译
macro_rules! foo {
    ($l:tt) => { bar!($l); }
}

macro_rules! bar {
    (3) => {}
}

foo!(3);
```

## 元变量

在匹配器中，`$`*名称*`:`*匹配段选择器* 这种句法格式匹配符合指定句法类型的 Rust 句法段，并将其绑定到元变量 `$`*名称* 上。有效的匹配段选择器包括：

  * `item`: [_程序项_][_Item_]
  * `block`: [_块表达式_][_BlockExpression_]
  * `stmt`: [_语句_][_Statement_]，注意此选择器不匹配句尾的分号（如果匹配器中提供了分号，会被当做分隔符），但碰到分号是自身的一部分的程序项语句的情况又会匹配。
  * `pat`: [_模式_][_PatternNoTopAlt_]
  * `expr`: [_表达式_][_Expression_]
  * `ty`: [_类型_][_Type_]
  * `ident`: [标识符或关键字][IDENTIFIER_OR_KEYWORD]
  * `path`: [_类型表达式_][_TypePath_] 形式的路径
  * `tt`: [_token树_][_TokenTree_]&nbsp;(单个 [token] 或宏匹配定界符 `()`、`[]` 或`{}` 中的标记)
  * `meta`: [_属性_][_Attr_]，属性中的内容
  * `lifetime`: [生存期token][LIFETIME_TOKEN]
  * `vis`: 可能为空的[_可见性_][_Visibility_]限定符
  * `literal`: 匹配 `-`<sup>?</sup>[_字面量表达式_][_LiteralExpression_]

因为匹配段类型已在匹配器中指定了，则在转码器中，元变量只简单地用 `$`*名称* 这种形式来指代就行了。元变量最终将被替换为跟它们匹配上的句法元素。元变量关键字 `$crate` 可以用来指代当前的 crate（请参阅后面的[卫生性(hygiene)][Hygiene]章节）。元变量可以被多次转码，也可以完全不转码。

## 重复元

在匹配器和转码器中，重复元被表示为：将需要重复的 token 放在 `$(`…`)` 内，然后后跟一个重复运算符(repetition operator)，这两者之间可以放置一个可选的分隔符(separator token)。分隔符可以是除定界符或重复运算符之外的任何 token，其中分号(`;`)和逗号(`,`)最常见。例如： `$( $i:ident ),*` 表示用逗号分隔的任何数量的标识符。嵌套的重复元是合法的。

重复运算符为：

- `*` — 表示任意数量的重复元。
- `+` — 表示至少有一个重复元。
- `?` — 表示一个可选的匹配段，可以出现零次或一次。

因为 `?` 表示最多出现一次，所以它不能与分隔符一起使用。

通过分隔符的分隔，重复的匹配段都会被匹配和转码为指定的数量的匹配段。元变量就和这些每个段中的重复元相匹配。例如，之前示例中的 `$( $i:ident ),*` 将 `$i` 去匹配列表中的所有标识符。

在转码过程中，重复元会受到额外的限制，以便于编译器知道该如何正确地扩展它们：

1.  在转码器中，元变量必须与它在匹配器中出现的次数、指示符类型以及其在重复元内的嵌套顺序都完全相同。因此，对于匹配器 `$( $i:ident ),*`，转码器 `=> { $i }`, `=> { $( $( $i)* )* }` 和 `=> { $( $i )+ }` 都是非法的，但是 `=> { $( $i );* }` 是正确的，它用分号分隔的标识符列表替换了逗号分隔的标识符列表。
2.  转码器中的每个重复元必须至少包含一个元变量，以便确定扩展多少次。如果在同一个重复元中出现多个元变量，则它们必须绑定到相同数量的匹配段上，不能有的多，有的少。例如，`( $( $i:ident ),* ; $( $j:ident ),* ) => (( $( ($i,$j) ),* ))` 里，绑定到 `$j` 的匹配段的数量必须与绑定到 `$i` 上的相同。这意味着用 `(a, b, c; d, e, f)` 调用这个宏是合法的，并且可扩展到 `((a,d), (b,e), (c,f))`，但是 `(a, b, c; d, e)` 是非法的，因为前后绑定的数量不同。此要求适用于嵌套的重复元的每一层。

## 作用域、导出以及导入

由于历史原因，声明宏的作用域并不完全像各种程序项那样工作。宏有两种形式的作用域：文本作用域(textual scope)和基于路径的作用域(path-based scope)。文本作用域基于宏在源文件中（定义和使用所）出现的顺序，或是跨多个源文件出现的顺序，文本作用域是默认的作用域。（后本节面将进一步解释这个。）基于路径的作用域与其他程序项作用域的运行方式相同。宏的作用域、导出和导入主要由其属性控制。

当声明宏被非限定标识符(unqualified identifier)（非多段路径段组成的限定性路径）调用时，会首先在文本作用域中查找。如果文本作用域中没有任何结果，则继续在基于路径的作用域中查找。如果宏的名称由路径限定，则只在基于路径的作用域中查找。

<!-- ignore: requires external crates -->
```rust,ignore
use lazy_static::lazy_static; // 基于路径的导入.

macro_rules! lazy_static { // 文本定义.
    (lazy) => {};
}

lazy_static!{lazy} // 首先通过文本作用域来查找我们的宏.
self::lazy_static!{} // 忽略文本作用域查找，直接使用基于路径的查找方式找到一个导入的宏.
```

### 文本作用域

文本作用域很大程度上取决于宏本身在源文件中的出现顺序，其工作方式与用 `let`语句声明的局部变量的作用域类似，只不过它可以直接位于模块下。当使用 `macro_rules!` 定义宏时，宏在定义之后进入其作用域（请注意，这不影响宏在定义中递归调用自己，因为宏调用的入口还是在定义之后的某次调用点上，此点开始的宏名称递归查找一定有效），在封闭它的作用域（通常是模块）结束时离开。文本作用域可以覆盖/进入子模块，甚至跨越多个文件：

<!-- ignore: requires external modules -->
```rust,ignore
//// src/lib.rs
mod has_macro {
    // m!{} // 报错: m 未在作用域内.

    macro_rules! m {
        () => {};
    }
    m!{} // OK: 在声明 m 后使用.

    mod uses_macro;
}

// m!{} // Error: m 未在作用域内.

//// src/has_macro/uses_macro.rs

m!{} // OK: m 在上层模块文件 src/lib.rs 中声明后使用
```

多次定义宏并不报错；除非超出作用域，否则最近的宏声明将屏蔽前一个。

```rust
macro_rules! m {
    (1) => {};
}

m!(1);

mod inner {
    m!(1);

    macro_rules! m {
        (2) => {};
    }
    // m!(1); // 报错: 没有设定规则来匹配 '1'
    m!(2);

    macro_rules! m {
        (3) => {};
    }
    m!(3);
}

m!(1);
```

宏也可以在函数内部声明和使用，其工作方式类似：

```rust
fn foo() {
    // m!(); // 报错: m 未在作用域内.
    macro_rules! m {
        () => {};
    }
    m!();
}


// m!(); // Error: m 未在作用域内.
```

### `macro_use`属性

*`macro_use`属性*有两种用途。首先，它可以通过作用于模块的方式让模块内的宏的作用域在模块关闭时不结束：

```rust
#[macro_use]
mod inner {
    macro_rules! m {
        () => {};
    }
}

m!();
```

其次，它可以用于从另一个 crate 里来导入宏，方法是将它附加到当前 crate 根模块中的 `extern crate` 声明前。以这种方式导入的宏会被导入到[`macro_use`预导入包][`macro_use` prelude]里，而不是直接文本导入，这意味着它们可以被任何其他同名宏屏蔽。虽然可以在导入语句之前使用 `#[macro_use]` 导入宏，但如果发生冲突，则最后导入的宏将胜出。可以使用可选的 [_MetaListIdents_]元项属性句法指定要导入的宏列表；当将 `#[macro_use]` 应用于模块上时，则不支持此指定操作。

<!-- ignore: requires external crates -->
```rust,ignore
#[macro_use(lazy_static)] // 或者使用 #[macro_use] 来导入所有宏.
extern crate lazy_static;

lazy_static!{}
// self::lazy_static!{} // 报错: lazy_static 没在 `self` 中定义
```

要用 `#[macro_use]` 导入宏必须先使用 `#[macro_export]` 导出，下文会有讲解。

### 基于路径的作用域

默认情况下，宏没有基于路径的作用域。但是如果该宏带有 `#[macro_export]` 属性，则相当于它在当前 crate 的根作用域的顶部被声明，它通常可以这样引用：

```rust
self::m!();
m!(); // OK: 基于路径的查找发现 m 在当前模块中有声明.

mod inner {
    super::m!();
    crate::m!();
}

mod mac {
    #[macro_export]
    macro_rules! m {
        () => {};
    }
}
```

标有 `#[macro_export]` 的宏始终是 `pub` 的，以便可以通过路径或前面所述的 `#[macro_use]` 方式让其他 crate 来引用。

## 卫生性

默认情况下，宏中引用的所有标识符都按原样展开，并在宏的调用位置上去查找。如果宏引用的程序项或宏不在调用位置的作用域内，则这可能会导致问题。为了解决这个问题，可以替代在路径的开头使用元变量 `$crate`，强制在定义宏的 crate 中进行查找。

<!-- ignore: requires external crates -->
```rust,ignore
//// 在 `helper_macro` crate 中.
#[macro_export]
macro_rules! helped {
    // () => { helper!() } // 这可能会导致错误，因为 'helper' 在当前作用域之后才定义.
    () => { $crate::helper!() }
}

#[macro_export]
macro_rules! helper {
    () => { () }
}

//// 在另一个 crate 中使用.
// 注意没有导入 `helper_macro::helper`!
use helper_macro::helped;

fn unit() {
    helped!();
}
```

请注意，由于 `$crate` 指的是当前的（`$crate` 源码出现的）crate，因此在引用非宏程序项时，它必须与全限定模块路径一起使用：

```rust
pub mod inner {
    #[macro_export]
    macro_rules! call_foo {
        () => { $crate::inner::foo() };
    }

    pub fn foo() {}
}
```

此外，尽管 `$crate` 允许宏在扩展时引用其自身 crate 中的程序项，但它的使用对可见性没有影响（，或者说它的使用仍受可见性条件的约束）。引用的程序项或宏必须仍然在调用位置处可见。在下面的示例中，任何试图从此 crate 外部调用 `call_foo!()` 的行为都将失败，因为 `foo()` 不是公有的。

```rust
#[macro_export]
macro_rules! call_foo {
    () => { $crate::foo() };
}

fn foo() {}
```
（译者注：原文给出的这个例子是能正常调用的，原文并没有给出在 crate 外部调用的例子）

> **版本&版次差异**：在 Rust 1.30 之前，`$crate` 和 `local_inner_macros`（后面会讲）不受支持。从该版本开始，它们与基于路径的宏导入（前面讲过）一起被添加进来，用以确保不需要在当前 crate 已经使用导入了某导出宏的情况下再额外手动导入此导出宏下面用到的辅助宏。如果要让 Rust 的早期版本编写的 crate 要使用辅助宏，需要修改为使用 `$crate` 或 `local_inner_macros`，以便与基于路径的导入一起工作。

当一个宏被导出时，可以在 `#[macro_export]` 属性里添加 `local_inner_macros` 属性值，用以自动为该属性修饰的宏内包含的所有宏调用自动添加 `$crate::` 前缀。这主要是作为一个工具来迁移那些在引入 `$crate` 之前的版本编写的 Rust 代码，以便它们能与 Rust 2018 版中基于路径的宏导入一起工作。在使用新版本编写的代码中不鼓励使用它。

```rust
#[macro_export(local_inner_macros)]
macro_rules! helped {
    () => { helper!() } // 自动转码为 $crate::helper!().
}

#[macro_export]
macro_rules! helper {
    () => { () }
}
```

## 随集歧义限制（译者注：该节还需要继续校对打磨，主要难点还是因为附录的宏定义规范译者还没有能全部搞懂）

宏系统使用的解析器相当强大，但是为了防止其在 Rust 的当前或未来版本中出现二义性解析，因此对它做出了限制。特别地，在消除二义性展开的基本规则之外又增加了一条：元变量匹配的非终结符(nonterminal)必须后跟一个已经确定为可以用来安全分隔匹配段的分隔符。

例如，像 `$i:expr [ , ]` 这样的宏匹配器在现今的 Rust 中理论上是可以接受的，因为现在 `[,]` 不可能是合法表达式的一部分，因此解析始终是明确的。但是，由于 `[` 可以开始一个尾随表达式(trailing expressions)，因此 `[` 不是一个可以安全排除在表达式后面出现的字符。如果在接下来的 Rust 版本中接受了 `[,]`，那么这个匹配器就会产生歧义或是错误解析，破坏正常代码。但是，像`$i:expr,` 或 `$i:expr;` 这样的匹配符始终是合法的，因为 `,` 和`;` 是合法的表达式分隔符。目前规范中的规则是：（译者注：下面的规则不是绝对的，因为宏的基础理论还在发展中。）

  * `expr` 和 `stmt` 只能后跟一个： `=>`、`,`、`;`。
  * `pat` 只能后跟一个： `=>`、`,`、`=`、`|`、`if`、`in`。
  * `path` 和 `ty` 只能后跟一个： `=>`、`,`、`=`、`|`、`;`、`:`、`>`、`>>`、`[`、`{`、`as`、`where`、块(`block`)型非终结符(block nonterminals)。
  * `vis` 只能后跟一个：`,`、非原生字符串 `priv` 以外的任何标识符和关键字、可以表示类型开始的任何 token、`ident`或`ty`或`path`型非终结符。    
    
    （译者注：可以表示类型开始的 token 有：{`(`, `[`, `!`, `\*`,`&`, `&&`, `?`, 生存期, `>`, `>>`, `::`, 非关键字标识符, `super`,`self`, `Self`, `extern`, `crate`, `$crate`, `_`, `for`, `impl`, `fn`, `unsafe`,`typeof`, `dyn`}。注意这个列表也不一定全。）
  
  * 其它所有的匹配段选择器没有限制。

当涉及到重复元时，随集歧义限制适用于所有可能的展开次数，注意需将重复元中的分隔符考虑在内。这意味着：

  * 如果重复元包含分隔符，则分隔符必须能够跟随重复元的内容重复。
  * 如果重复元可以重复多次（`*` 或 `+`），那么重复元的内容必须能自我重复。
  * 重复元前后内容必须严格匹配匹配器中指定的前后内容。
  * 如果重复元可以匹配零次（`*` 或 `?`），那么它后面的内容必须能够直接跟在它前面的内容后面。
  
有关更多详细信息，请参阅[正式规范]。

[Hygiene]: #hygiene
[IDENTIFIER]: identifiers.md
[IDENTIFIER_OR_KEYWORD]: identifiers.md
[LIFETIME_TOKEN]: tokens.md#lifetimes-and-loop-labels
[Metavariables]: #metavariables
[Repetitions]: #repetitions
[_Attr_]: attributes.md
[_BlockExpression_]: expressions/block-expr.md
[_DelimTokenTree_]: macros.md
[_Expression_]: expressions.md
[_Item_]: items.md
[_LiteralExpression_]: expressions/literal-expr.md
[_MetaListIdents_]: attributes.md#meta-item-attribute-syntax
[_PatternNoTopAlt_]: patterns.md
[_Statement_]: statements.md
[_TokenTree_]: macros.md#macro-invocation
[_Token_]: tokens.md
[_TypePath_]: paths.md#paths-in-types
[_Type_]: types.md#type-expressions
[_Visibility_]: visibility-and-privacy.md
[formal specification]: macro-ambiguity.md
[token]: tokens.md
[`macro_use` prelude]: names/preludes.md#macro_use-prelude
