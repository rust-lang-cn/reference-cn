# 记号

记号是语法中的主要组合，使用正则（非递归）语言定义。Rust 源输入可以被分解为下述类型的记号：

* [关键字][Keywords]
* [标识符][identifier]
* [字面量](#字面量)
* [生命周期](#生命周期)
* [运算符及符号](#运算符及符号)
* [分隔符](#分隔符)

本书语法中，“简单”记号采用[字符串表组合][string
table production]形式，并以`等宽（monospace）`字体显示。

[string table production]: notation.md#string-table-productions

## 字面量

字面量是一个包含单独记号（而不是一串记号）的表达式，它立即且直接的表示了它所赋的值，而不是通过名字或其它赋值规则引用它。字面量是[常量表达式](const_eval.md#constant-expressions)的一种形式，所以它（主要）在编译时赋值。

### 示例

#### 字符和字符串

| | 示例 | `#` 集合 | 字符集 | 转义 |
|-|------|---------|--------|-----|
| [字符](#字符字面量) | `'H'` | 0  | 全部 Unicode | [引号](#引号转义) & [ASCII](#ascii-转义) & [Unicode](#unicode-转义) |
| [字符串](#字符串字面量) | `"hello"` | 0 | 全部 Unicode | [引号](#引号转义) & [ASCII](#ascii-转义) & [Unicode](#unicode-转义) |
| [原生字符串](#原生字符串字面量) | `r#"hello"#` | 0 ... | 全部 Unicode | `N/A` |
| [字节](#字节字面量) | `b'H'` | 0 | 全部 ASCII   | [引号](#引号转义) & [字节](#字节转义)  |
| [字节串](#字节串字面量) | `b"hello"` | 0 | 全部 ASCII   | [引号](#引号转义) & [字节](#字节转义) |
| [原生字节串](#原生字节串字面量) | `br#"hello"#` | 0 ... | 全部 ASCII   | `N/A` |

\* 字面量两侧的 `#` 数量必须相同。

#### ASCII 转义

|   | 名称 |
|---|------|
| `\x41` | 7 位字符编码（2位，最大值为 `0x7F`） |
| `\n` | 换行符 |
| `\r` | 回车符 |
| `\t` | 制表符 |
| `\\` | 反斜线 |
| `\0` | Null/空/零值（译者注：Rust 中没有 Null） |

#### 字节转义

|   | 名称 |
|---|------|
| `\x7F` | 8 位字符编码（2位） |
| `\n` | 换行符 |
| `\r` | 回车符 |
| `\t` | 制表符 |
| `\\` | 反斜线 |
| `\0` | Null/空/零值（译者注：Rust 中没有 Null） |

#### Unicode 转义

|   | 名称 |
|---|------|
| `\u{7FFF}` | 24 位 Unicode 字符编码（最多6个数字） |

#### 引号转义

|   | Name |
|---|------|
| `\'` | 单引号 |
| `\"` | 双引号 |

#### 数值

| [数字字面量](#数字字面量)`*` | 示例 | 幂 | 后缀 |
|---------------------------|-------|----|-----|
| 十进制整数 | `98_222` | `N/A` | 整数后缀 |
| 十六进制整数 | `0xff` | `N/A` | 整数后缀 |
| 二进制整数 | `0o77` | `N/A` | 整数后缀 |
| 二进制整数 | `0b1111_0000` | `N/A` | 整数后缀 |
| 浮点数 | `123.0E+77` | `Optional` | 浮点数后缀 |

`*` 所有数字字面量允许 `_` 作为可视分隔符：`1_234.0E+18f64`

#### 后缀

后缀是紧跟（无空格）字面量主体部分之后的非原生标识符。

具有任意后缀的任何类型的字面量（如字符串、整数等）都是合法记号，可以传递给宏而不会产生错误。宏本身将决定如何诠释此类记号，以及是否产生错误。

```rust
macro_rules! blackhole { ($tt:tt) => () }

blackhole!("string"suffix); // 没毛病 :-)
```

但是，解析为 Rust 代码的字面量记号，其后缀是受限制的。对于非数字字面量记号，将拒绝任何后缀；而对于数字字面量，仅接受具有下表中的后缀。

| 整型 | 浮点型 |
|------|-------|
| `u8`, `i8`, `u16`, `i16`, `u32`, `i32`, `u64`, `i64`, `u128`, `i128`, `usize`, `isize` | `f32`, `f64` |

### 字符和字符串字面量

#### 字符字面量

> **<sup>Lexer</sup>**\
> CHAR_LITERAL :\
> &nbsp;&nbsp; `'` ( ~\[`'` `\` \\n \\r \\t] | QUOTE_ESCAPE | ASCII_ESCAPE | UNICODE_ESCAPE ) `'`
>
> QUOTE_ESCAPE :\
> &nbsp;&nbsp; `\'` | `\"`
>
> ASCII_ESCAPE :\
> &nbsp;&nbsp; &nbsp;&nbsp; `\x` OCT_DIGIT HEX_DIGIT\
> &nbsp;&nbsp; | `\n` | `\r` | `\t` | `\\` | `\0`
>
> UNICODE_ESCAPE :\
> &nbsp;&nbsp; `\u{` ( HEX_DIGIT `_`<sup>\*</sup> )<sup>1..6</sup> `}`

_字符字面量_ 是位于两个 `U+0027`（单引号）字符内的单个 Unicode 字符。当它是 `U+0027` 自身时，须前置 _转义_ 字符 `U+005C`（`\`）。

#### 字符串字面量

> **<sup>Lexer</sup>**\
> STRING_LITERAL :\
> &nbsp;&nbsp; `"` (\
> &nbsp;&nbsp; &nbsp;&nbsp; ~\[`"` `\` _IsolatedCR_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | QUOTE_ESCAPE\
> &nbsp;&nbsp; &nbsp;&nbsp; | ASCII_ESCAPE\
> &nbsp;&nbsp; &nbsp;&nbsp; | UNICODE_ESCAPE\
> &nbsp;&nbsp; &nbsp;&nbsp; | STRING_CONTINUE\
> &nbsp;&nbsp; )<sup>\*</sup> `"`
>
> STRING_CONTINUE :\
> &nbsp;&nbsp; `\` _followed by_ \\n

_字符串字面量_ 是位于两个 `U+0022`（双引号）字符内的任意 Unicode 字符序列。当它是 `U+0022` 自身时，须前置 _转义_ 字符 `U+005C`（`\`）。

字符串字面量中允许分行。分行符可以换行符（`U+000A`），也可以是回车符和一对换行符（`U+000D`, `U+000A`）。此对字节序列通常转换为 `U+000A`，但有例外：当分行符前置一个未转义的 `U+005C` 字符（`\`）时，将会导致 `U+005C` 字符、换行符和下一行开头的所有空白都被忽略。是故下述示例中，`a` 和 `b` 是等同的：

```rust
let a = "foobar";
let b = "foo\
         bar";

assert_eq!(a,b);
```

#### 字符转义

不管是字符字面量，还是非原生字符串字面量，都有一些额外 _转义_。一个转义以一个 `U+005C`（`\`）开始，并后跟如下形式之一：

* _7 位代码点转义_ 以 `U+0078`（`x`）开头，后紧跟两位 _十六进制数字_，最大值为 `0x7F`。表示其值等于它所提供的十六进制 ASCII 字符，但不能确定其是 Unicode 代码点还是字节值，所以更大的值是不被允许的。
* _24 位代码点转义_ 以 `U+0075`（`u`）开头，后跟多达六位 _十六进制数字_，位于大括号 `U+007B`（`{`）和 `U+007D`（`}`）之间。表示其值等于它所提供的十六进制 Unicode 代码点。
* _空白转义_ 是字符 `U+006E`（`n`），`U+0072`（`r`），或者 `U+0074`（`t`）之一，依次表示 Unicode 值 `U+000A`（LF），`U+000D`（CR），或者 `U+0009`（HT）。
* _null/空/零值转义_ 是字符 `U+0030`（`0`），表示 Unicode 值 `U+0000`（NUL）。
* _反斜杠转义_ 是字符 `U+005C`（`\`），必须转义才能表示其自身。

#### 原生字符串字面量

> **<sup>Lexer</sup>**\
> RAW_STRING_LITERAL :\
> &nbsp;&nbsp; `r` RAW_STRING_CONTENT
>
> RAW_STRING_CONTENT :\
> &nbsp;&nbsp; &nbsp;&nbsp; `"` ( ~ _IsolatedCR_ )<sup>* (non-greedy)</sup> `"`\
> &nbsp;&nbsp; | `#` RAW_STRING_CONTENT `#`

原生字符串字面量不处理任何转义。它以字符 `U+0072`（`r`）后跟零个或多个字符 `U+0023`（`#`），以及一个 `U+0022`（双引号）字符开始。_原生字符串正文_ 可包含任意 Unicode 字符序列，并仅以另一个 `U+0022`（双引号）字符结尾，后跟与开头的 `U+0022`（双引号）字符前同等数量的 `U+0023`（`#`）字符。

所有包含在原生字符串正文中的 Unicode 字符都代表他们自身，字符 `U+0022`（双引号）（当后跟的零个或多个 `U+0023`（`#`）字符用于开始原生字符串字面量时除外）或 `U+005C`（`\`）并无特殊含义。

字符串字面量示例：

```rust
"foo"; r"foo";                     // foo
"\"foo\""; r#""foo""#;             // "foo"

"foo #\"# bar";
r##"foo #"# bar"##;                // foo #"# bar

"\x52"; "R"; r"R";                 // R
"\\x52"; r"\x52";                  // \x52
```

### 字节和字节串字面量

#### 字节字面量

> **<sup>Lexer</sup>**\
> BYTE_LITERAL :\
> &nbsp;&nbsp; `b'` ( ASCII_FOR_CHAR | BYTE_ESCAPE )  `'`
>
> ASCII_FOR_CHAR :\
> &nbsp;&nbsp; _any ASCII (i.e. 0x00 to 0x7F), except_ `'`, `\`, \\n, \\r or \\t
>
> BYTE_ESCAPE :\
> &nbsp;&nbsp; &nbsp;&nbsp; `\x` HEX_DIGIT HEX_DIGIT\
> &nbsp;&nbsp; | `\n` | `\r` | `\t` | `\\` | `\0`

_字节字面量_ 是单个 ASCII 字符（在 `U+0000` 到 `U+007F` 范围内）或单个 _转义符_，其前置字符 `U+0062`（`b`）和 `U+0027`（单引号），后跟字符 `U+0027`。如果字面量中存在字符 `U+0027`，则必须由前置字符 `U+005C`（`\`）转义，它相当于一个 `u8`（无符号 8 位整型）_数字字面量_。

#### 字节串字面量

> **<sup>Lexer</sup>**\
> BYTE_STRING_LITERAL :\
> &nbsp;&nbsp; `b"` ( ASCII_FOR_STRING | BYTE_ESCAPE | STRING_CONTINUE )<sup>\*</sup> `"`
>
> ASCII_FOR_STRING :\
> &nbsp;&nbsp; _any ASCII (i.e 0x00 to 0x7F), except_ `"`, `\` _and IsolatedCR_

非原生 _字节串字面量_ 是 ASCII 字符和 _转义符_，前置字符`U+0062`（`b`）和 `U+0022`（双引号），以字符 `U+0022` 结尾。若字符 `U+0022` 出现在字节串字面量中，须由前置字符 `U+005C`（`\`）_转义_。或者，字节串字面量可以是 _原生字节串字面量_，定义为：长度为 `n` 的字节串字面量类型是 `&'static [u8; n]`。

一些额外的 _转义_ 可以在字节或非原生字节串字面量中使用，转义以 `U+005C`（`\`）开始，并后跟如下形式之一：

* _字节转义_ 以 `U+0078`（`x`）开始，后跟两个十六进制数，表示十六进制值代表的字节。
* _空白转义_ 是字符 `U+006E`（`n`）、`U+0072`（`r`），或 `U+0074`（`t`）之一，分别表示字节值 `0x0A`（ASCII LF）、`0x0D`（ASCII CR），或 `0x09`（ASCII HT）。
* _null/空/零值转义_ 是字符 `U+0030`（`0`），表示字节值 `U+0000`（ASCII NUL）。
* _反斜杠转义_ 是字符 `U+005C`（`\`），必须被转义以表示其 ASCII 编码 `0x5C`。

#### 原生字节串字面量

> **<sup>Lexer</sup>**\
> RAW_BYTE_STRING_LITERAL :\
> &nbsp;&nbsp; `br` RAW_BYTE_STRING_CONTENT
>
> RAW_BYTE_STRING_CONTENT :\
> &nbsp;&nbsp; &nbsp;&nbsp; `"` ASCII<sup>* (non-greedy)</sup> `"`\
> &nbsp;&nbsp; | `#` RAW_STRING_CONTENT `#`
>
> ASCII :\
> &nbsp;&nbsp; _any ASCII (i.e. 0x00 to 0x7F)_

原生字节串字面量不处理任何转义。它们以字符 `U+0062`（`b`）开头，后跟 `U+0072`（`r`），后跟零个或多个字符 `U+0023`（`#`）及 `U+0022`（双引号）字符。_原生字节串正文_ 可包含任意 ASCII 字符序列，并仅以另一个 `U+0022`（双引号）字符结尾，后面与开头 `U+0022`（双引号）字符之前同等数量的 `U+0023`（`#`）字符。原生字节串字面量不能包含任何非 ASCII 字节。

原生字节串正文中的所有字符表示其 ASCII 编码，字符 `U+0022`（双引号）（当后跟的零个或多个 `U+0023`（`#`）字符用于开始原生字节串字面量时除外）或 `U+005C`（`\`）并无特殊含义。

字节串字面量示例：

```rust
b"foo"; br"foo";                     // foo
b"\"foo\""; br#""foo""#;             // "foo"

b"foo #\"# bar";
br##"foo #"# bar"##;                 // foo #"# bar

b"\x52"; b"R"; br"R";                // R
b"\\x52"; br"\x52";                  // \x52
```

### 数字字面量

_数字字面量_ 可以是 _整型字面量_，也可以是 _浮点型字面量_，此两类字面量的辨别语法是混合的。

#### 整型字面量

> **<sup>Lexer</sup>**\
> INTEGER_LITERAL :\
> &nbsp;&nbsp; ( DEC_LITERAL | BIN_LITERAL | OCT_LITERAL | HEX_LITERAL )
>              INTEGER_SUFFIX<sup>?</sup>
>
> DEC_LITERAL :\
> &nbsp;&nbsp; DEC_DIGIT (DEC_DIGIT|`_`)<sup>\*</sup>
>
> TUPLE_INDEX :\
> &nbsp;&nbsp; &nbsp;&nbsp; `0`
> &nbsp;&nbsp; | NON_ZERO_DEC_DIGIT DEC_DIGIT<sup>\*</sup>
>
> BIN_LITERAL :\
> &nbsp;&nbsp; `0b` (BIN_DIGIT|`_`)<sup>\*</sup> BIN_DIGIT (BIN_DIGIT|`_`)<sup>\*</sup>
>
> OCT_LITERAL :\
> &nbsp;&nbsp; `0o` (OCT_DIGIT|`_`)<sup>\*</sup> OCT_DIGIT (OCT_DIGIT|`_`)<sup>\*</sup>
>
> HEX_LITERAL :\
> &nbsp;&nbsp; `0x` (HEX_DIGIT|`_`)<sup>\*</sup> HEX_DIGIT (HEX_DIGIT|`_`)<sup>\*</sup>
>
\> BIN_DIGIT : \[`0`-`1`]
>
> OCT_DIGIT : \[`0`-`7`]
>
> DEC_DIGIT : \[`0`-`9`]
>
> NON_ZERO_DEC_DIGIT : \[`1`-`9`]
>
> HEX_DIGIT : \[`0`-`9` `a`-`f` `A`-`F`]
>
> INTEGER_SUFFIX :\
> &nbsp;&nbsp; &nbsp;&nbsp; `u8` | `u16` | `u32` | `u64` | `u128` | `usize`\
> &nbsp;&nbsp; | `i8` | `i16` | `i32` | `i64` | `i128` | `isize`

整型字面量具备下述 4 种形式之一：

* _十进制字面量_ 以 _十进制数_ 开头，后跟 _十进制数_ 和 _下划线_ 的任意组合。
* _元组索引_ 可以是 `0`；也可以以 _非零十进制数_ 开始，其后跟零个或多个十进制数。元组索引用于引用[元组][tuples]，[元组结构体][tuple structs]，以及[元组变量][tuple variants]中的字段。
* _十六进制字面量_ 以字符序列 `U+0030` `U+0078`（`0x`）开头，后跟十六进制数和下划线的任意组合（至少一个数字）。
* _八进制字面量_ 以字符序列 `U+0030` `U+006F`（`0o`）开头，后跟八进制数和下划线的任意组合（至少一个数字）。
* _二进制字面量_ 以字符序列 `U+0030` `U+0062`（`0b`）开头，后跟二进制数和下划线的任意组合（至少一个数字）。

如同其它字面量，整型字面量后面（紧跟，不带空白）可跟 _整型后缀_，该后缀强制设定了字面量的类型。整型后缀须为如下整型类型之一：`u8`、`i8`、`u16`、`i16`、`u32`、`i32`、`u64`、`i64`、`u128`、`i128`、`usize` 或 `isize`。

_无后缀_ 整型字面量的类型通过类型推断确定：

* 如果整型类型可以通过程序上下文 _唯一_ 确定，则无后缀整型字面量即为该类型。
* 如果程序上下文对类型做了约束，则默认为有符号 32 位整型 `i32`。
* 如果程序上下文过度限制了类型，则将其视为静态类型错误。

不同形式的整型字面量示例：

```rust
123;                               // type i32
123i32;                            // type i32
123u32;                            // type u32
123_u32;                           // type u32
let a: u64 = 123;                  // type u64

0xff;                              // type i32
0xff_u8;                           // type u8

0o70;                              // type i32
0o70_i16;                          // type i16

0b1111_1111_1001_0000;             // type i32
0b1111_1111_1001_0000i64;          // type i64
0b________1;                       // type i32

0usize;                            // type usize
```

无效整型字面量示例：

```rust,compile_fail
// 无效后缀

0invalidSuffix;

// 错误进制

123AFB43;
0b0102;
0o0581;

// 类型溢出

128_i8;
256_u8;

// 至少需要一个数字（二进制、十六进制、八进制）

0b_;
0b____;
```

注意：Rust 语法将 `-1i8` 视作[算术取负运算符][unary minus
operator]对整型字面量 `1i8` 的应用，而非单个整型字面量。

[unary minus operator]: expressions/operator-expr.md#negation-operators

#### 浮点型字面量

> **<sup>Lexer</sup>**\
> FLOAT_LITERAL :\
> &nbsp;&nbsp; &nbsp;&nbsp; DEC_LITERAL `.`
>   _(not immediately followed by `.`, `_` or an [identifier]_)\
> &nbsp;&nbsp; | DEC_LITERAL FLOAT_EXPONENT\
> &nbsp;&nbsp; | DEC_LITERAL `.` DEC_LITERAL FLOAT_EXPONENT<sup>?</sup>\
> &nbsp;&nbsp; | DEC_LITERAL (`.` DEC_LITERAL)<sup>?</sup>
>                    FLOAT_EXPONENT<sup>?</sup> FLOAT_SUFFIX
>
> FLOAT_EXPONENT :\
> &nbsp;&nbsp; (`e`|`E`) (`+`|`-`)?
>               (DEC_DIGIT|`_`)<sup>\*</sup> DEC_DIGIT (DEC_DIGIT|`_`)<sup>\*</sup>
>
> FLOAT_SUFFIX :\
> &nbsp;&nbsp; `f32` | `f64`

浮点型字面量具有如下两种形式之一：

* _十进制字面量_ 后跟一个句点字符 `U+002E`（`.`），之后可选后跟另一个十进制字面量和 _指数_。
* 单个 _十进制字面量_，后跟 _指数_。

如同整型字面量，浮点型字面量也可后跟一个后缀，只要后缀之前部分不以 `U+002E`（`.`）结尾。后缀强制设定了字面量类型。有两种有效的 _浮点型后缀_，`f32` 和 `f64`（32 位和 64 位浮点类型），它们显式地确定了字面量类型。

_无后缀_ 浮点型字面量的类型通过类型推断确定：

* 如果浮点型类型可以通过程序上下文 _唯一_ 确定，则无后缀浮点型字面量即为该类型。
* 如果程序上下文对类型做了约束，则默认为 `f64`。
* 如果程序上下文过度限制了类型，则将其视为静态类型错误。

不同形式的浮点型字面量示例：

```rust
123.0f64;        // type f64
0.1f64;          // type f64
0.1f32;          // type f32
12E+99_f64;      // type f64
let x: f64 = 2.; // type f64
```

最后一个例子稍显不同，因为不能对一个以句点结尾的浮点型字面量使用后缀语法，`2.f64` 才会尝试在 `2` 上调用名为 `f64` 的方法。

浮点数所代表的语义在[“机器类型”]["Machine Types"]中描述。

["Machine Types"]: types/numeric.md

### 布尔型字面量

> **<sup>Lexer</sup>**\
> BOOLEAN_LITERAL :\
> &nbsp;&nbsp; &nbsp;&nbsp; `true`\
> &nbsp;&nbsp; | `false`

布尔类型有两个值：`true` 和 `false`。

## 生命周期

> **<sup>Lexer</sup>**\
> LIFETIME_TOKEN :\
> &nbsp;&nbsp; &nbsp;&nbsp; `'` [IDENTIFIER_OR_KEYWORD][identifier]\
> &nbsp;&nbsp; | `'_`
>
> LIFETIME_OR_LABEL :\
> &nbsp;&nbsp; &nbsp;&nbsp; `'` [NON_KEYWORD_IDENTIFIER][identifier]

生命周期参数和[循环标签][loop labels] LIFETIME_OR_LABEL 记号。词法上接受任何 LIFETIME_TOKEN，比如在宏中使用。

[loop labels]: expressions/loop-expr.md

## 运算符及符号

如下是 Rust 语言运算符的完整列表，它们各自的用法和意义在链接页面中定义。

| Symbol | Name        | Usage |
|--------|-------------|-------|
| `+`    | 加/正号（Plus） | [算术加法][arith]、[复合类型限制][Trait Bounds]、[宏匹配器][macros]
| `-`    | 减/负号（Minus） | [算术减法][arith]、[算术取负][Negation]
| `*`    | 星号（Star） | [算术乘法][arith]、[解引用][Dereference]、[裸指针][Raw Pointers]、[宏匹配器][macros]
| `/`    | 斜线（Slash） | [算术除法][arith]
| `%`    | 百分号（Percent） | [算术取模][arith]
| `^`    | 脱字符（Caret） | [按位异或][arith]
| `!`    | 叹号（Not）         | [按位非、逻辑非][negation]、[宏调用（展开）][macros]、[内部属性][attributes]、[never 型][Never Type]
| `&`    | 与（And） | [按位与、逻辑与][arith]、[借用][Borrow]、[引用][References]、[引用模式][Reference patterns]
| <code>\|</code> | 或（Or） | [按位或、逻辑或][arith]、[闭包][Closures]、[匹配][Match]
| `&&`   | AndAnd | [Lazy 与][lazy-bool]、[借用][Borrow]、[引用][References]、[引用模式][Reference patterns]
| <code>\|\|</code> | OrOr | [Lazy 或][lazy-bool]、[闭包][Closures]
| `<<`   | Shl | [左移][arith]、[嵌套泛型][generics]
| `>>`   | Shr | [右移][arith], [嵌套泛型][generics]
| `+=`   | PlusEq      | [算术加法及赋值][compound]
| `-=`   | MinusEq     | [算术减法及赋值][compound]
| `*=`   | StarEq      | [算术乘法及赋值][compound]
| `/=`   | SlashEq     | [算术除法及赋值][compound]
| `%=`   | PercentEq   | [算术取模及赋值][compound]
| `^=`   | CaretEq     | [按位异或及赋值][compound]
| `&=`   | AndEq       | [按位与及赋值][compound]
| <code>\|=</code> | OrEq | [按位或及赋值][compound]
| `<<=`  | ShlEq       | [左移及赋值][compound]
| `>>=`  | ShrEq       | [右移及赋值][compound]、[嵌套泛型][generics]
| `=`    | 等号（Eq） | [赋值][Assignment]、[属性][Attributes]、类型定义
| `==`   | EqEq        | [等于][comparison]
| `!=`   | Ne          | [不等于][comparison]
| `>`    | Gt          | [大于][comparison]、[泛型][Generics]、[路径][Paths]
| `<`    | Lt          | [小于][comparison]、[泛型][Generics]、[路径][Paths]
| `>=`   | Ge          | [大于等于][comparison]、[泛型][Generics]
| `<=`   | Le          | [小于等于][comparison]
| `@`    | At          | [模式绑定][Subpattern binding]
| `_`    | 下划线（Underscore） | [通配符模式][Wildcard patterns]、[类型推导][Inferred types]
| `.`    | 点号（Dot） | [成员访问][field]、[元组索引][Tuple index]
| `..`   | DotDot      | [范围][range]、[结构体表达式][Struct expressions]、[模式][Patterns]
| `...`  | DotDotDot   | [变长参数函数][extern]、[范围模式][Range patterns]
| `..=`  | DotDotEq    | [闭区间][range]、[范围模式][Range patterns]
| `,`    | 逗号（Comma） | 参数以及元素分隔符
| `;`    | 分号（Semi） | 各类项和语句结束符、[数组类型][Array types]
| `:`    | Colon       | 参数以及元素分隔符
| `::`   | PathSep     | [路径分隔符][paths]
| `->`   | RArrow      | [函数返回类型][functions]、[闭包返回类型][closures]
| `=>`   | FatArrow    | [匹配分支][match]、[宏][Macros]
| `#`    | 井号（Pound） | [属性][Attributes]
| `$`    | 美元符（Dollar） | [宏][Macros]
| `?`    | 问号（Question） | [问号运算符][question], [不定大小][sized], [宏匹配器][macros]

## 分隔符

括号用于语法的各个部分，且总是成对出现。括号及其内的记号在[宏][macros]中被称作“记号树”。括号有三种类型：

|   括号   | 类型            |
|---------|-----------------|
| `{` `}` | 花/大括号        |
| `[` `]` | 方/中括号        |
| `(` `)` | 圆/小括号        |


[Inferred types]: types/inferred.md
[Range patterns]: patterns.md#range-patterns
[Reference patterns]: patterns.md#reference-patterns
[Subpattern binding]: patterns.md#identifier-patterns
[Wildcard patterns]: patterns.md#wildcard-pattern
[arith]: expressions/operator-expr.md#arithmetic-and-logical-binary-operators
[array types]: types/array.md
[assignment]: expressions/operator-expr.md#assignment-expressions
[attributes]: attributes.md
[borrow]: expressions/operator-expr.md#borrow-operators
[closures]: expressions/closure-expr.md
[comparison]: expressions/operator-expr.md#comparison-operators
[compound]: expressions/operator-expr.md#compound-assignment-expressions
[dereference]: expressions/operator-expr.md#the-dereference-operator
[extern]: items/external-blocks.md
[field]: expressions/field-expr.md
[functions]: items/functions.md
[generics]: items/generics.md
[identifier]: identifiers.md
[keywords]: keywords.md
[lazy-bool]: expressions/operator-expr.md#lazy-boolean-operators
[macros]: macros-by-example.md
[match]: expressions/match-expr.md
[negation]: expressions/operator-expr.md#negation-operators
[never type]: types/never.md
[paths]: paths.md
[patterns]: patterns.md
[question]: expressions/operator-expr.md#the-question-mark-operator
[range]: expressions/range-expr.md
[raw pointers]: types/pointer.md#raw-pointers-const-and-mut
[references]: types/pointer.md
[sized]: trait-bounds.md#sized
[struct expressions]: expressions/struct-expr.md
[trait bounds]: trait-bounds.md
[tuple index]: expressions/tuple-expr.md#tuple-indexing-expressions
[tuple structs]: items/structs.md
[tuple variants]: items/enumerations.md
[tuples]: types/tuple.md
