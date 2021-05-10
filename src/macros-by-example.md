# 声明宏

> **<sup>Syntax</sup>**\
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
> &nbsp;&nbsp; &nbsp;&nbsp; [_Token_]<sub>_except $ and delimiters_</sub>\
> &nbsp;&nbsp; | _MacroMatcher_\
> &nbsp;&nbsp; | `$` [IDENTIFIER] `:` _MacroFragSpec_\
> &nbsp;&nbsp; | `$` `(` _MacroMatch_<sup>+</sup> `)` _MacroRepSep_<sup>?</sup> _MacroRepOp_
>
> _MacroFragSpec_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `block` | `expr` | `ident` | `item` | `lifetime` | `literal`\
> &nbsp;&nbsp; | `meta` | `pat` | `path` | `stmt` | `tt` | `ty` | `vis`
>
> _MacroRepSep_ :\
> &nbsp;&nbsp; [_Token_]<sub>_except delimiters and repetition operators_</sub>
>
> _MacroRepOp_ :\
> &nbsp;&nbsp; `*` | `+` | `?`
>
> _MacroTranscriber_ :\
> &nbsp;&nbsp; [_DelimTokenTree_]

`声明宏`允许用户以声明的方式定义语法扩展，我们称这种扩展为“示例宏（macros by example）”，或简单的称作“宏（macros）”。

每个声明宏有其名字，以及一个或多个 _规则_。每个规则包含两个部分：一个是 _匹配器_——描述声明宏的匹配语法；另一个是 _转换器_——描述声明宏被成功匹配和调用后的替换语法。匹配器和转换器都必须由分隔符包围。宏可以扩展为表达式、语句、项（包括 trait、实现，以及外部项）、类型，或者模式。

## 转换

当宏被调用时，声明宏的扩展程序根据名称查找宏调用，并依次尝试每个宏规则。声明宏会根据第一个成功的匹配进行转换；即使转换结果导致错误，也不会尝试后续的匹配。声明宏执行匹配时，不执行预判；如果编译器无法确定如何解析一个标记的宏调用，则会导致错误。下述示例中，编译器不会预判传入的标识符以查看其跟随记号是否为“`)`”——尽管预判将允许编译器明确地解析调用：

```rust,compile_fail
macro_rules! ambiguity {
    ($($i:ident)* $j:ident) => { };
}

ambiguity!(error); // Error: local ambiguity
```

在匹配器和转换器中，`$` 记号用于调用宏引擎中的特殊行为（下文的[元变量][Metavariables]和[重复][Repetitions]中有详述）。不属于这种调用的记号是按字面意思匹配和转换的——只有一个例外：匹配器的外部分隔符将匹配任何一对分隔符。因此，比如匹配器 `(())` 将匹配 `{()}` 而不是 `{{}}`。字符 `$` 不能被匹配或按照字面意义转换。

当将匹配片段发送到另一个声明宏时，第二个宏中的匹配器将看到此匹配片段类型的不完全抽象语法树（AST）。第二个宏不能根据记号的字面意义来匹配匹配器中的片段，只能使用同一类型的片段分类符。匹配片段的类型 `ident`、`lifetime`、`tt` 是例外状况，_可以_ 通过记号的字面意义进行匹配。如下通过例子阐述上述限制：

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

下述例子说明如何在匹配 `tt` 片段后直接匹配记号：

```rust
// compiles OK
macro_rules! foo {
    ($l:tt) => { bar!($l); }
}

macro_rules! bar {
    (3) => {}
}

foo!(3);
```

## 元变量

在匹配器中，`$`_名字_ `:` _片段分类符_ 匹配指定类型的 Rust 语法片段，并将其绑定到元变量 `$`_名字_。有效的片段分类符是：

  * `item`：[项][_Item_]
  * `block`：[块表达式][_BlockExpression_]
  * `stmt`：末尾没有分号的[语句][_Statement_]（需要分号的项语句除外）
  * `pat`：[模式][_Pattern_]
  * `expr`：[表达式][_Expression_]
  * `ty`：[类型][_Type_]
  * `ident`：[标识符/关键字][IDENTIFIER_OR_KEYWORD]
  * `path`：[类型路径][_TypePath_]风格的路径
  * `tt`：[记号树][_TokenTree_]&nbsp;（匹配分隔符 `()`、`[]`、`{}` 中的单个或多个[记号][token])
  * `meta`：[属性][_Attr_]，属性的内容
  * `lifetime`：[生命周期标记][LIFETIME_TOKEN]
  * `vis`：可能为空的[可见性][_Visibility_]限定符
  * `literal`：匹配“`-`”<sup>?</sup>[字面量表达式][_LiteralExpression_]


在转换器中，因为片段类型是在匹配器中指定的，所以元变量仅被 `$`_名称_ 简单引用。元变量将被替换为与片段类型相匹配的语法元素；元变量关键字 `$crate` 可用于引用当前 crate（详细参见[辅助][Hygiene]）；元变量可以被多次转换，也可以根本不转换。 

## 重复

在匹配器和转换器中，通过将要重复的记号放在 `$(`…`)` 内来代表重复，可选后跟一个分隔记号，然后后跟重复运算符。分隔记号可以是除分隔符或某个重复运算符之外的任何记号，分号 `;` 和逗号 `,` 是最常用的。例如：`$( $i:ident ),*` 表示用逗号分隔的任何数量的标识符，且允许嵌套式重复。 

重复运算符包括：

- `*` — 代表任意数量的重复——即 `0` 次或`多`次重复。
- `+` — 代表至少 `1` 次的重复。
- `?` — 代表出现 `0` 次或者 `1` 次的可选片段。

因为 `?` 最多代表出现一个匹配项，所以不能和分隔符一起使用。

重复的片段可匹配并转换为指定数量的片段，由分隔符分隔。元变量被匹配到与其相符的每个重复匹配。例如：上述示例 `$( $i:ident
),*` 将匹配 `$i` 与列表中的所有标识符相符。

在转换过程中，会有附加的限制用于重复操作，以便于编译器知道如何正确地展开重复的记号：

1. 首先，元变量在转换器中出现的数量、种类，以及重复的嵌套顺序，与其在匹配器中出现的完全相同。所以若匹配器中出现 `$( $i:ident ),*`，那么转换器中出现的 `=> { $i }`、`=> { $( $( $i)* )* }`，以及 `=> { $( $i )+ }` 都是不合法的。而 `=> { $( $i );* }` 是正确的，并用分号分隔的列表替换逗号分隔的标识符列表。
1. 其次，转换器中的每个重复则必须包含至少一个元变量，从而决定其扩展的次数。如果在同一个重复中出现多个元变量，则它们必须被绑定到相同数量的片段上。例如：`( $( $i:ident ),* ; $( $j:ident ),* ) =>
    ( $( ($i,$j) ),*` 必须绑定到与 `$i` 片段同等数量的 `$j` 片段，这意味着使用 `(a, b, c; d, e, f)` 调用宏是合法的，并且可扩展为 `((a,d)、(b,e)、(c,f))`。但是 `(a, b, c; d, e)` 不合法的，因为其数量不同。此要求适用于嵌套重复的每一层。

## 作用域、导出，以及导入

由于历史原因，声明宏的作用域并不完全像其它 Rust 数据项那样工作。声明宏有两种形式的作用域：文本作用域和基于路径的作用域。文本作用域是默认的作用域规则，基于源文件中代码出现的顺序，甚至是在多个文件中出现的顺序——下文将进一步解释文本作用域。基于路径的作用域的工作方式与其它 Rust 数据项作用域的工作方式完全相同。声明宏中，作用域、导出，以及导入主要由属性控制。

当声明宏被非限定标识符（不是多重路径的一部分）调用时，首先在文本作用域中查找。如果文本作用域中没有任何结果，则继续在基于路径的作用域中查找。如果声明宏的名称限定为路径，则仅在基于路径的作用域中查找。

<!-- ignore: requires external crates -->
```rust,ignore
use lazy_static::lazy_static; // 基于路径的导入

macro_rules! lazy_static { // 文本定义
    (lazy) => {};
}

lazy_static!{lazy} // 首先，文本查找，发现声明宏
self::lazy_static!{} // 基于路径的查找，则忽略文本定义的宏，找到导入的宏
```

### 文本作用域

文本作用域主要基于源代码中声明的顺序，其工作方式与使用 `let` 声明的局部变量的作用域类似，只不过它也适用于模块级。当使用 `macro_rules!` 定义宏时，宏在定义之后进入其作用域（注意，由于名称是从调用位置查找的，因此它仍然可以递归使用），直到其周围的作用域——通常为模块——关闭为止。文本作用域可应用于子模块，甚至涵盖多个文件：

<!-- ignore: requires external modules -->
```rust,ignore
//// src/lib.rs
mod has_macro {
    // m!{} // Error: m is not in scope.

    macro_rules! m {
        () => {};
    }
    m!{} // OK: appears after declaration of m.

    mod uses_macro;
}

// m!{} // Error: m is not in scope.

//// src/has_macro/uses_macro.rs

m!{} // OK: appears after declaration of m in src/lib.rs
```

可以多次或重复定义一个宏，这并无不妥；除非超出作用域，否则最新的宏声明将遮蔽以前的宏声明。

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
    // m!(1); // Error: no rule matches '1'
    m!(2);

    macro_rules! m {
        (3) => {};
    }
    m!(3);
}

m!(1);
```

宏也可以局部声明和使用在函数内部，工作方式类似：

```rust
fn foo() {
    // m!(); // Error: m is not in scope.
    macro_rules! m {
        () => {};
    }
    m!();
}


// m!(); // Error: m is not in scope.
```

### `macro_use` 属性

*`macro_use` 属性*有两个用途。首先，它可以用于在模块关闭时，使模块的宏作用域不结束，方法是将 `macro_use` 应用于模块：

```rust
#[macro_use]
mod inner {
    macro_rules! m {
        () => {};
    }
}

m!();
```

其次，它可以用于从另一个 crate 导入宏，方法是在 crate 根模块中，将 `macro_use` 附加到 `extern crate` 声明。以这种方式导入的宏会被导入到 crate 的预处理中，而不是文本导入，这意味着它们可以被任何其他名称遮蔽。虽然可以在导入语句之前使用 `#[macro_use]` 导入宏，但如果发生冲突，则最后导入的宏将被应用。可选地，可以使用 [_MetaListIdents_] 语法指定要导入的宏列表；`#[macro_use]` 应用于模块时，则不支持此操作。

<!-- ignore: requires external crates -->
```rust,ignore
#[macro_use(lazy_static)] // Or #[macro_use] to import all macros.
extern crate lazy_static;

lazy_static!{}
// self::lazy_static!{} // Error: lazy_static is not defined in `self`
```

使用 `#[macro_use]` 导入的宏必须使用 `#[macro_export]` 导出，下文详述。

### 基于路径的作用域

默认情况下，宏没有基于路径的作用域。但是，如果它具有 `#[macro_export]` 属性，那么它被声明在 crate 根作用域，并且通常可以被引用。如下所示：

```rust
self::m!();
m!(); // OK: Path-based lookup finds m in the current module.

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

标记为 `#[macro_export]` 的宏始终是 `pub`，并且可以由其他 crate 引用。如上文所述：可以通过路径或通过 `#[macro_use]` 来引用。

## 卫生（Hygiene）

默认情况下，宏中引用的所有标识符都按原样展开，并在宏的调用位置查找。如果宏引用的项或宏不在调用位置的作用域内，则可能会导致问题。为了缓解这种情况，可以在路径的开头使用 `$crate` 元变量，以强制在定义宏的 crate 内部进行查找。

<!-- ignore: requires external crates -->
```rust,ignore
//// 宏定义在 `helper_macro` crate 中
#[macro_export]
macro_rules! helped {
    // () => { helper!() } // This might lead to an error due to 'helper' not being in scope.
    () => { $crate::helper!() }
}

#[macro_export]
macro_rules! helper {
    () => { () }
}

//// 在其它 crate 使用
// Note that `helper_macro::helper` is not imported!
use helper_macro::helped;

fn unit() {
    helped!();
}
```

请注意，由于 `$crate` 引用了当前的 crate，因此在引用非宏项时，它必须与完全限定的模块路径一起使用： 

```rust
pub mod inner {
    #[macro_export]
    macro_rules! call_foo {
        () => { $crate::inner::foo() };
    }

    pub fn foo() {}
}
```

此外，尽管 `$crate` 允许宏在扩展时引用其自身 crate 中的项目，但它的使用对可见性没有影响。引用的项或宏，仍然必须从调用位置可见。在下面的例子中，任何试图从其 crate 外部调用 `call_foo!()` 都将失败，因为 `foo()` 不是公有的。

```rust
#[macro_export]
macro_rules! call_foo {
    () => { $crate::foo() };
}

fn foo() {}
```

> **版本/版次差异**：在 Rust 1.30 之前，`$crate` 和 `local_inner_macros` 不受支持。它们与基于路径的宏导入（如上所述）一起添加，以确保辅助宏不需要由宏导出 crate 的用户手动导入。为 Rust 早期版本编写的 crate 要使用辅助宏，需要修改为使用 `$crate` 或者 `local_inner_macros`，以便与基于路径的导入一起工作。

导出宏时，`#[macro_export]` 属性可以具有
`local_inner_macros` 关键字，以自动为包含的所有宏调用添加  `$crate::` 前缀。这主要是作为一个工具，来移植在 `$crate` 添加到 Rust 语言之前所编写的代码，以便于与 Rust 2018 的基于路径的宏导入一起工作。在新版本/版次的代码中不鼓励使用它。

```rust
#[macro_export(local_inner_macros)]
macro_rules! helped {
    () => { helper!() } // Automatically converted to $crate::helper!().
}

#[macro_export]
macro_rules! helper {
    () => { () }
}
```

## 遵循的歧义限制

宏系统使用的解析器相当强大，但是为了防止当前或将来版本的语言出现歧义，它受到了限制。特别是，除了关于歧义性展开的规则外，由元变量匹配的非终结符，后面必须跟有一个已确定可以在这种匹配之后安全使用的标记。

例如，像 `$i:expr [ , ]` 这样的宏匹配器，在现今的 Rust 中理论上是可以接受的，因为 `[,]` 不能是合法表达式的一部分，因此解析总是清晰的。但是，因为 `[` 可以开始尾随表达式，`[` 不是一个可以安全排除在表达式后面的字符。如果在 Rust 的更高版本中接受了 `[,]`，那么这个匹配器就会产生歧义或是无法正确解析，破坏了工作代码。但是，像 `$i:expr,` 或者 `$i:expr;` 这样的匹配符是合法的，因为 `,` 和
`;` 是合法的表达式分隔符。具体规则是：

  * `expr` 和 `stmt` 后面只可以跟随一个：`=>`，`,` 或者 `;`。
  * `pat` 后面只可以跟随一个：`=>`，`,`，`=`，`|`，`if` 或者 `in`。
  * `path` 和 `ty` 后面只可以跟随一个：`=>`，`,`，`=`，`|`，`;`，`:`，`>`，`>>`，`[`，`{`，`as`，`where` 或者一个`块`片段说明符的宏变量。
  * `vis` 后面只可以跟随一个：`,`，一个非原生的标识符 `priv`，任何可以用 `ident`，`ty` 或者`路径`片段说明符开始的类型或元变量的标记。
  * 其他所有的片段说明符没有限制。

当涉及到重复时，规则适用于所有可能的展开，同时考虑到分隔符。这意味着：

  * 如果重复包含分隔符，则该分隔符必须能够跟随重复的内容。
  * 如果重复可以重复多次（`*` 或者 `+`），那么内容必须能够遵循自身。
  * 重复的内容必须能够遵循前面的内容，之后的内容必须能够遵循重复的内容。
  * 如果重复能够匹配零次（`*` 或者 `?`），那么后面的内容必须能够遵循前面的内容。

有关更多详细信息，请参阅[正式规范][formal specification]。

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
[_Pattern_]: patterns.md
[_Statement_]: statements.md
[_TokenTree_]: macros.md#macro-invocation
[_Token_]: tokens.md
[_TypePath_]: paths.md#paths-in-types
[_Type_]: types.md#type-expressions
[_Visibility_]: visibility-and-privacy.md
[formal specification]: macro-ambiguity.md
[token]: tokens.md
