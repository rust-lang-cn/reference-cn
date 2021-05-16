# 模式

>[patterns.md](https://github.com/rust-lang/reference/blob/master/src/patterns.md)\
>commit: 100fa060ce1d2db7e4d1533efd289f9c18d08d53 \
>本章译文最后维护日期：2021-5-6

> **<sup>句法</sup>**\
> _Pattern_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `|`<sup>?</sup> _PatternNoTopAlt_  ( `|` _PatternNoTopAlt_ )<sup>\*</sup>
>
> _PatternNoTopAlt_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _PatternWithoutRange_\
> &nbsp;&nbsp; | [_RangePattern_]
>
> _PatternWithoutRange_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_LiteralPattern_]\
> &nbsp;&nbsp; | [_IdentifierPattern_]\
> &nbsp;&nbsp; | [_WildcardPattern_]\
> &nbsp;&nbsp; | [_RestPattern_]\
> &nbsp;&nbsp; | [_ObsoleteRangePattern_]\
> &nbsp;&nbsp; | [_ReferencePattern_]\
> &nbsp;&nbsp; | [_StructPattern_]\
> &nbsp;&nbsp; | [_TupleStructPattern_]\
> &nbsp;&nbsp; | [_TuplePattern_]\
> &nbsp;&nbsp; | [_GroupedPattern_]\
> &nbsp;&nbsp; | [_SlicePattern_]\
> &nbsp;&nbsp; | [_PathPattern_]\
> &nbsp;&nbsp; | [_MacroInvocation_]

模式基于给定数据结构去匹配值，并可选地将变量和这些结构中匹配到的值绑定起来。模式也用在变量声明上和函数（包括闭包）的参数上。

下面示例中的模式完成四件事：

* 测试 `person` 是否在其 `car`字段中填充了内容。
* 测试 `person` 的 `age`字段（的值）是否在 13 到 19 之间，并将其值绑定到给定的变量 `person_age` 上。
* 将对 `name`字段的引用绑定到给定变量 `person_name` 上。
* 忽略 `person` 的其余字段。其余字段可以有任何值，并且不会绑定到任何变量上。

```rust
# struct Car;
# struct Computer;
# struct Person {
#     name: String,
#     car: Option<Car>,
#     computer: Option<Computer>,
#     age: u8,
# }
# let person = Person {
#     name: String::from("John"),
#     car: Some(Car),
#     computer: None,
#     age: 15,
# };
if let
    Person {
        car: Some(_),
        age: person_age @ 13..=19,
        name: ref person_name,
        ..
    } = person
{
    println!("{} has a car and is {} years old.", person_name, person_age);
}
```

模式用于：

* [`let`声明](statements.md#let语句)
* [函数](items/functions.md)和[闭包](expressions/closure-expr.md)的参数。
* [匹配(`match`)表达式](expressions/match-expr.md)
* [`if let`表达式](expressions/if-expr.md)
* [`while let`表达式](expressions/loop-expr.md#predicate-pattern-loops)
* [`for`表达式](expressions/loop-expr.md#iterator-loops)

## 解构

模式可用于*解构*[结构体(`struct`)][structs]、[枚举(`enum`)][enums]和[元组][tuples]。解构将一个值分解成它的组件组成，使用的句法与创建此类值时的几乎相同。在[检验对象][scrutinee]表达式的类型为结构体(`struct`)、枚举(`enum`)或元组(`tuple`)的模式中，占位符(`_`) 代表*一个*数据字段，而通配符 `..` 代表特定变量/变体(variant)的*所有*剩余字段。当使用字段的名称（而不是字段序号）来解构数据结构时，允许将 `fieldname` 当作 `fieldname: fieldname` 的简写形式书写。

```rust
# enum Message {
#     Quit,
#     WriteString(String),
#     Move { x: i32, y: i32 },
#     ChangeColor(u8, u8, u8),
# }
# let message = Message::Quit;
match message {
    Message::Quit => println!("Quit"),
    Message::WriteString(write) => println!("{}", &write),
    Message::Move{ x, y: 0 } => println!("move {} horizontally", x),
    Message::Move{ .. } => println!("other move"),
    Message::ChangeColor { 0: red, 1: green, 2: _ } => {
        println!("color change, red: {}, green: {}", red, green);
    }
};
```

## 可反驳性

当一个模式有可能与它所匹配的值不匹配时，我们就说它是*可反驳型的(refutable)*。也就是说，*不可反驳型(irrefutable)*模式总是能与它们所匹配的值匹配成功。例如：

```rust
let (x, y) = (1, 2);               // "(x, y)" 是一个不可反驳型模式

if let (a, 3) = (1, 2) {           // "(a, 3)" 是可反驳型的, 将不会匹配
    panic!("Shouldn't reach here");
} else if let (a, 4) = (3, 4) {    // "(a, 4)" 是可反驳型的, 将会匹配
    println!("Matched ({}, 4)", a);
}
```

## 字面量模式

> **<sup>句法</sup>**\
> _LiteralPattern_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [BOOLEAN_LITERAL]\
> &nbsp;&nbsp; | [CHAR_LITERAL]\
> &nbsp;&nbsp; | [BYTE_LITERAL]\
> &nbsp;&nbsp; | [STRING_LITERAL]\
> &nbsp;&nbsp; | [RAW_STRING_LITERAL]\
> &nbsp;&nbsp; | [BYTE_STRING_LITERAL]\
> &nbsp;&nbsp; | [RAW_BYTE_STRING_LITERAL]\
> &nbsp;&nbsp; | `-`<sup>?</sup> [INTEGER_LITERAL]\
> &nbsp;&nbsp; | `-`<sup>?</sup> [FLOAT_LITERAL]

[BOOLEAN_LITERAL]: tokens.md#boolean-literals
[CHAR_LITERAL]: tokens.md#character-literals
[BYTE_LITERAL]: tokens.md#byte-literals
[STRING_LITERAL]: tokens.md#string-literals
[RAW_STRING_LITERAL]: tokens.md#raw-string-literals
[BYTE_STRING_LITERAL]: tokens.md#byte-string-literals
[RAW_BYTE_STRING_LITERAL]: tokens.md#raw-byte-string-literals
[INTEGER_LITERAL]: tokens.md#integer-literals
[FLOAT_LITERAL]: tokens.md#floating-point-literals

*字面量模式*匹配的值与字面量所创建的值完全相同。由于负数不是[字面量][literals]，（特设定）字面量模式也接受字面量前的可选负号，它的作用类似于否定运算符。

<div class="warning">

浮点字面量目前还可以使用，但是由于它们在数值比较时带来的复杂性，在将来的 Rust 版本中，它们将被禁止用于字面量模式(参见 [issue #41620](https://github.com/rust-lang/rust/issues/41620))。

</div>

字面量模式总是可以反驳型的。

示例：

```rust
for i in -2..5 {
    match i {
        -1 => println!("It's minus one"),
        1 => println!("It's a one"),
        2|4 => println!("It's either a two or a four"),
        _ => println!("Matched none of the arms"),
    }
}
```

## 标识符模式

> **<sup>句法</sup>**\
> _IdentifierPattern_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `ref`<sup>?</sup> `mut`<sup>?</sup> [IDENTIFIER] (`@` [_Pattern_] ) <sup>?</sup>

标识符模式将它们匹配的值绑定到一个变量上。此标识符在该模式中必须是唯一的。该变量会在作用域中遮蔽任何同名的变量。这种绑定的作用域取决于使用模式的上下文（例如 `let`绑定或匹配臂(`match` arm)[^译注1]）。

标识符模式只能包含一个标识符（也可能前带一个 `mut`），能匹配任何值，并将其绑定到该标识符上。最常见的标识符模式应用场景就是用在变量声明上和用在函数（包括闭包）的参数上。

```rust
let mut variable = 10;
fn sum(x: i32, y: i32) -> i32 {
#    x + y
# }
```

要将模式匹配到的值绑定到变量上，也可使用句法 `variable @ subpattern`。例如，下面示例中将值 2 绑定到 `e` 上（不是整个区间(range)：这里的区间是一个区间子模式(range subpattern)）。

```rust
let x = 2;

match x {
    e @ 1 ..= 5 => println!("got a range element {}", e),
    _ => println!("anything"),
}
```

默认情况下，标识符模式里变量会和匹配到的值的一个拷贝副本绑定，或匹配值自身移动过来和变量完成绑定，具体是使用拷贝语义还是移动语义取决于匹配到的值是否实现了 [`Copy`]。也可以通过使用关键字 `ref` 将变量和值的引用绑定，或者使用 `ref mut` 将变量和值的可变引用绑定。示例：

```rust
# let a = Some(10);
match a {
    None => (),
    Some(value) => (),
}

match a {
    None => (),
    Some(ref value) => (),
}
```

在第一个匹配表达式中，值被拷贝（或移动）（到变量 `value` 上）。在第二个匹配中，对相同内存位置的引用被绑定到变量上。之所以需要这种句法，是因为在解构子模式(destructuring subpatterns)里，操作符 `&` 不能应用在值的字段上。例如，以下内容无效：

```rust,compile_fail
# struct Person {
#    name: String,
#    age: u8,
# }
# let value = Person { name: String::from("John"), age: 23 };
if let Person { name: &person_name, age: 18..=150 } = value { }
```

要使其有效，请按如下方式编写代码：

```rust
# struct Person {
#    name: String,
#    age: u8,
# }
# let value = Person { name: String::from("John"), age: 23 };
if let Person { name: ref person_name, age: 18..=150 } = value { }
```

这里，`ref` 不是被匹配的一部分。这里它唯一的目的就是使变量和匹配值的引用绑定起来，而不是潜在地拷贝或移动匹配到的内容。

[路径模式(Path pattern)](#path-patterns)优先于标识符模式。如果给某个标识符指定了 `ref` 或 `ref mut`，同时该标识符又遮蔽了某个常量，这会导致错误。

如果 `@`子模式是不可反驳型的或未指定子模式，则标识符模式是不可反驳型的。

### 绑定方式

基于人类工程学的考虑，为了让引用和匹配值的绑定更容易一些，模式会自动选择不同的*绑定方式*。当引用值与非引用模式匹配时，这将自动地被视为 `ref` 或 `ref mut` 绑定方式。示例：

```rust
let x: &Option<i32> = &Some(3);
if let Some(y) = x {
    // y 被转换为`ref y` ，其类型为 &i32
}
```

*非引用模式(Non-reference patterns)*包括**除**上面这种绑定模式和后面会讲到的[通配符模式](#wildcard-pattern)（`_`）、匹配引用类型的[常量(`const`)模式](#path-patterns)和[引用模式](#reference-patterns)这些模式以外的所有模式。

如果绑定模式(binding pattern)中没有显式地包含 `ref`、`ref mut`、`mut`，那么它将使用*默认绑定方式*来确定如何绑定变量。默认绑定方式以使用移动语义的“移动(move)”方式开始。当匹配一个模式时，编译器对模式从外到内逐层匹配。每次非引用模式和引用匹配上了时，引用都会自动解引用出最后的值，并更新默认绑定方式，再进行最终的匹配。此时引用会将默认绑定方式设置为 `ref` 方式。可变引用会将模式设置为 `ref mut` 方式，除非绑定方式已经是 `ref` 了（在这种情况下它仍然是 `ref` 方式）。如果自动解引用解出的值仍然是引用，则会重复解引用。[^译注2]

移动语义的绑定方式和引用语义的绑定方式可以在同一个模式中混合使用，这样做会导致绑定对象的部分被移走，并且之后无法再使用该对象。这只适用于类型无法拷贝的情况下。

下面的示例中，`name` 被移出了 `person`，因此如果再试图把 `person` 作为一个整体使用，或再次使用 `person.name`，将会因为*部分移出(partial move)*的问题而报错。

示例：

```rust
# struct Person {
#    name: String,
#    age: u8,
# }
# let person = Person{ name: String::from("John"), age: 23 };
// 在 `age` 被引用绑定的情况下，`name` 被从 person 中移出
let Person { name, ref age } = person;
```

## 通配符模式

> **<sup>句法</sup>**\
> _WildcardPattern_ :\
> &nbsp;&nbsp; `_`

*通配符模式*（下划线符号）能与任何值匹配。常用它来忽略那些无关紧要的值。在其他模式中使用该模式时，它匹配单个数据字段（与和匹配所有其余字段的 `..` 相对）。与标识符模式不同，它不会复制、移动或借用它匹配的值。

示例：

```rust
# let x = 20;
let (a, _) = (10, x);   // x 一定会被 _ 匹配上
# assert_eq!(a, 10);

// 忽略一个函数/闭包参数
let real_part = |a: f64, _: f64| { a };

// 忽略结构体的一个字段
# struct RGBA {
#    r: f32,
#    g: f32,
#    b: f32,
#    a: f32,
# }
# let color = RGBA{r: 0.4, g: 0.1, b: 0.9, a: 0.5};
let RGBA{r: red, g: green, b: blue, a: _} = color;
# assert_eq!(color.r, red);
# assert_eq!(color.g, green);
# assert_eq!(color.b, blue);

// 能接收带任何值的任何 Some
# let x = Some(10);
if let Some(_) = x {}
```

通配符模式总是不可反驳型的。

## 剩余模式

> **<sup>句法</sup>**\
> _RestPattern_ :\
> &nbsp;&nbsp; `..`

*剩余模式*（`..` token）充当匹配长度可变的模式(variable-length pattern)，它匹配之前之后没有匹配的零个或多个元素。它只能在[元组](#tuple-patterns)模式、[元组结构体](#tuple-struct-patterns)模式和[切片](#slice-patterns)模式中使用，并且只能作为这些模式中的一个元素出现一次。当作为[标识符模式](#identifier-patterns)的子模式时，它也可出现在[切片模式](#slice-patterns)里。

剩余模式总是不可反驳型的。

示例：

```rust
# let words = vec!["a", "b", "c"];
# let slice = &words[..];
match slice {
    [] => println!("slice is empty"),
    [one] => println!("single element {}", one),
    [head, tail @ ..] => println!("head={} tail={:?}", head, tail),
}

match slice {
    // 忽略除最后一个元素以外的所有元素，并且最后一个元素必须是 "!".
    [.., "!"] => println!("!!!"),

    // `start` 是除最后一个元素之外的所有元素的一个切片，最后一个元素必须是 “z”。
    [start @ .., "z"] => println!("starts with: {:?}", start),

    // `end` 是除第一个元素之外的所有元素的一个切片，第一个元素必须是 “a”
    ["a", end @ ..] => println!("ends with: {:?}", end),

    rest => println!("{:?}", rest),
}

if let [.., penultimate, _] = slice {
    println!("next to last is {}", penultimate);
}

# let tuple = (1, 2, 3, 4, 5);
// 剩余模式也可是在元组和元组结构体模式中使用。
match tuple {
    (1, .., y, z) => println!("y={} z={}", y, z),
    (.., 5) => println!("tail must be 5"),
    (..) => println!("matches everything else"),
}
```

## 区间模式

> **<sup>句法</sup>**\
> _RangePattern_ :\
> &nbsp;&nbsp; _RangePatternBound_ `..=` _RangePatternBound_
>
> _ObsoleteRangePattern_ :(译者注：废弃的区间模式句法/产生式) \ 
> &nbsp;&nbsp; _RangePatternBound_ `...` _RangePatternBound_
>
> _RangePatternBound_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [CHAR_LITERAL]\
> &nbsp;&nbsp; | [BYTE_LITERAL]\
> &nbsp;&nbsp; | `-`<sup>?</sup> [INTEGER_LITERAL]\
> &nbsp;&nbsp; | `-`<sup>?</sup> [FLOAT_LITERAL]\
> &nbsp;&nbsp; | [_PathInExpression_]\
> &nbsp;&nbsp; | [_QualifiedPathInExpression_]

区间模式匹配在其上下边界定义的封闭区间内的值。例如，一个模式 `'m'..='p'` 将只能匹配值`'m'`、`'n'`、`'o'` 和 `'p'`。它的边界值可以是字面量，也可以是指向常量值的路径。

一个模式 a `..=` b 必须总是有 a &le; b。例如，`10..=0` 这样的区间模式是错误的。

保留 `...`句法只是为了向后兼容。

区间模式只适用于标量类型(scalar type)。可接受的类型有：

* 整型（u8、i8、u16、i16、usize、isize ...）。
* 字符型（char）。
* 浮点类型（f32 和 f64）。这已被弃用，在未来版本的 Rust 中将不可用（参见 [issue #41620](https://github.com/rust-lang/rust/issues/41620)）。

示例：

```rust
# let c = 'f';
let valid_variable = match c {
    'a'..='z' => true,
    'A'..='Z' => true,
    'α'..='ω' => true,
    _ => false,
};

# let ph = 10;
println!("{}", match ph {
    0..=6 => "acid",
    7 => "neutral",
    8..=14 => "base",
    _ => unreachable!(),
});

// 使用指向常量值的路径：
# const TROPOSPHERE_MIN : u8 = 6;
# const TROPOSPHERE_MAX : u8 = 20;
#
# const STRATOSPHERE_MIN : u8 = TROPOSPHERE_MAX + 1;
# const STRATOSPHERE_MAX : u8 = 50;
#
# const MESOSPHERE_MIN : u8 = STRATOSPHERE_MAX + 1;
# const MESOSPHERE_MAX : u8 = 85;
#
# let altitude = 70;
#
println!("{}", match altitude {
    TROPOSPHERE_MIN..=TROPOSPHERE_MAX => "troposphere",
    STRATOSPHERE_MIN..=STRATOSPHERE_MAX => "stratosphere",
    MESOSPHERE_MIN..=MESOSPHERE_MAX => "mesosphere",
    _ => "outer space, maybe",
});

# pub mod binary {
#     pub const MEGA : u64 = 1024*1024;
#     pub const GIGA : u64 = 1024*1024*1024;
# }
# let n_items = 20_832_425;
# let bytes_per_item = 12;
if let size @ binary::MEGA..=binary::GIGA = n_items * bytes_per_item {
    println!("这适用并占用{}个字节", size);
}

# trait MaxValue {
#     const MAX: u64;
# }
# impl MaxValue for u8 {
#     const MAX: u64 = (1 << 8) - 1;
# }
# impl MaxValue for u16 {
#     const MAX: u64 = (1 << 16) - 1;
# }
# impl MaxValue for u32 {
#     const MAX: u64 = (1 << 32) - 1;
# }
// 使用限定路径：
println!("{}", match 0xfacade {
    0 ..= <u8 as MaxValue>::MAX => "fits in a u8",
    0 ..= <u16 as MaxValue>::MAX => "fits in a u16",
    0 ..= <u32 as MaxValue>::MAX => "fits in a u32",
    _ => "too big",
});
```

当区间模式匹配某（非usize 和 非isize）整型类型和字符型(`char`)的整个值域时，此模式是不可反驳型的。例如，`0u8..=255u8` 是不可反驳型的。某类整型的值区间是从该类型的最小值到该类型最大值的闭区间。字符型(`char`)的值的区间就是那些包含所有 Unicode 标量值的区间，即 `'\u{0000}'..='\u{D7FF}'` 和 `'\u{E000}'..='\u{10FFFF}'`。

## 引用模式

> **<sup>句法</sup>**\
> _ReferencePattern_ :\
> &nbsp;&nbsp; (`&`|`&&`) `mut`<sup>?</sup> [_PatternWithoutRange_]

引用模式对当前匹配的指针做解引用，从而能借用它们：

例如，下面 `x: &i32` 上的两个匹配是等效的：

```rust
let int_reference = &3;

let a = match *int_reference { 0 => "zero", _ => "some" };
let b = match int_reference { &0 => "zero", _ => "some" };

assert_eq!(a, b);
```

引用模式的文法产生式(grammar production)要求必须使用 token `&&` 来匹配对引用的引用，因为 `&&` 本身是一个单独的 token，而不是两个 `&` token。

>译者注：举例
>```
>let a = Some(&&10);
>match a {
>    Some( &&value ) => println!("{}", value),
>    None => {}
>}
>```

引用模式中添加关键字 `mut` 可对可变引用做解引用。引用模式中的可变性标记必须与作为匹配对象的那个引用的可变性匹配。

引用模式总是不可反驳型的。

## 结构体模式

> **<sup>句法</sup>**\
> _StructPattern_ :\
> &nbsp;&nbsp; [_PathInExpression_] `{`\
> &nbsp;&nbsp; &nbsp;&nbsp; _StructPatternElements_ <sup>?</sup>\
> &nbsp;&nbsp; `}`
>
> _StructPatternElements_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _StructPatternFields_ (`,` | `,` _StructPatternEtCetera_)<sup>?</sup>\
> &nbsp;&nbsp; | _StructPatternEtCetera_
>
> _StructPatternFields_ :\
> &nbsp;&nbsp; _StructPatternField_ (`,` _StructPatternField_) <sup>\*</sup>
>
> _StructPatternField_ :\
> &nbsp;&nbsp; [_OuterAttribute_] <sup>\*</sup>\
> &nbsp;&nbsp; (\
> &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; [TUPLE_INDEX] `:` [_Pattern_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [IDENTIFIER] `:` [_Pattern_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | `ref`<sup>?</sup> `mut`<sup>?</sup> [IDENTIFIER]\
> &nbsp;&nbsp; )
>
> _StructPatternEtCetera_ :\
> &nbsp;&nbsp; [_OuterAttribute_] <sup>\*</sup>\
> &nbsp;&nbsp; `..`

[_OuterAttribute_]: attributes.md
[TUPLE_INDEX]: tokens.md#tuple-index

结构体模式匹配与子模式定义的所有条件匹配的结构体值。它也被用来[解构](#destructuring)结构体。

在结构体模式中，结构体字段需通过名称、索引（对于元组结构体来说）来指代，或者通过使用 `..` 来忽略：

```rust
# struct Point {
#     x: u32,
#     y: u32,
# }
# let s = Point {x: 1, y: 1};
#
match s {
    Point {x: 10, y: 20} => (),
    Point {y: 10, x: 20} => (),    // 顺序没关系
    Point {x: 10, ..} => (),
    Point {..} => (),
}

# struct PointTuple (
#     u32,
#     u32,
# );
# let t = PointTuple(1, 2);
#
match t {
    PointTuple {0: 10, 1: 20} => (),
    PointTuple {1: 10, 0: 20} => (),   // 顺序没关系
    PointTuple {0: 10, ..} => (),
    PointTuple {..} => (),
}
```

如果没使用 `..`，需要提供所有字段的详尽匹配：

```rust
# struct Struct {
#    a: i32,
#    b: char,
#    c: bool,
# }
# let mut struct_value = Struct{a: 10, b: 'X', c: false};
#
match struct_value {
    Struct{a: 10, b: 'X', c: false} => (),
    Struct{a: 10, b: 'X', ref c} => (),
    Struct{a: 10, b: 'X', ref mut c} => (),
    Struct{a: 10, b: 'X', c: _} => (),
    Struct{a: _, b: _, c: _} => (),
}
```

_`ref` 和/或 `mut` IDENTIFIER_ 这样的句法格式能匹配任意值，并将其绑定到与给定字段同名的变量上。

```rust
# struct Struct {
#    a: i32,
#    b: char,
#    c: bool,
# }
# let struct_value = Struct{a: 10, b: 'X', c: false};
#
let Struct{a: x, b: y, c: z} = struct_value;          // 解构所有的字段
```

当一个结构体模式的子模式是可反驳型的，那这个结构体模式就是可反驳型的。

## 元组结构体模式

> **<sup>句法</sup>**\
> _TupleStructPattern_ :\
> &nbsp;&nbsp; [_PathInExpression_] `(` _TupleStructItems_<sup>?</sup> `)`
>
> _TupleStructItems_ :\
> &nbsp;&nbsp; [_Pattern_]&nbsp;( `,` [_Pattern_] )<sup>\*</sup> `,`<sup>?</sup>

元组结构体模式匹配元组结构体值和枚举值，这些值将与该模式的子模式定义的所有条件进行匹配。它还被用于[解构](#destructuring)元组结构体值或枚举值。

当元组结构体模式的一个子模式是可反驳型的，则该元组结构体模式就是可反驳型的。

## 元组模式

> **<sup>句法</sup>**\
> _TuplePattern_ :\
> &nbsp;&nbsp; `(` _TuplePatternItems_<sup>?</sup> `)`
>
> _TuplePatternItems_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_Pattern_] `,`\
> &nbsp;&nbsp; | [_RestPattern_]\
> &nbsp;&nbsp; | [_Pattern_]&nbsp;(`,` [_Pattern_])<sup>+</sup> `,`<sup>?</sup>

元组模式匹配与子模式定义的所有条件匹配的元组值。它们还被用来[解构](#destructuring)元组值。

内部只带有一个[剩余模式][_RestPattern_](_RestPattern_)的元组句法形式 `(..)` 是一种内部不需要逗号分割的特殊匹配形式，它可以匹配任意长度的元组。

当元组模式的一个子模式是可反驳型的，那该元组模式就是可反驳型的。

使用元组模式的示例：

```rust
let pair = (10, "ten");
let (a, b) = pair;

assert_eq!(a, 10);
assert_eq!(b, "ten");
```

## 分组模式

> **<sup>句法</sup>**\
> _GroupedPattern_ :\
> &nbsp;&nbsp; `(` [_Pattern_] `)`

将模式括在圆括号内可用来显式控制复合模式的优先级。例如，像 `&0..=5` 这样的引用模式和区间模式相邻就会引起歧义，这时可以用圆括号来消除歧义。

```rust
let int_reference = &3;
match int_reference {
    &(0..=5) => (),
    _ => (),
}
```

## 切片模式

> **<sup>句法</sup>**\
> _SlicePattern_ :\
> &nbsp;&nbsp; `[` _SlicePatternItems_<sup>?</sup> `]`
>
> _SlicePatternItems_ :\
> &nbsp;&nbsp; [_Pattern_] \(`,` [_Pattern_])<sup>\*</sup> `,`<sup>?</sup>

切片模式可以匹配固定长度的数组和动态长度的切片。

```rust
// 固定长度
let arr = [1, 2, 3];
match arr {
    [1, _, _] => "从 1 开始",
    [a, b, c] => "从其他值开始",
};
```
```rust
// 动态长度
let v = vec![1, 2, 3];
match v[..] {
    [a, b] => { /* 这个匹配臂不适用，因为长度不匹配 */ }
    [a, b, c] => { /* 这个匹配臂适用 */ }
    _ => { /* 这个通配符是必需的，因为长度不是编译时可知的 */ }
};
```

在匹配数组时，只要每个元素是不可反驳型的，切片模式就是不可反驳型的。当匹配切片时，只有单个 `..` [剩余模式](#rest-patterns)或带有 `..`（剩余模式）作为子模式的[标识符模式](#identifier-patterns)的情况才是不可反驳型的。

## 路径模式

> **<sup>句法</sup>**\
> _PathPattern_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_PathInExpression_]\
> &nbsp;&nbsp; | [_QualifiedPathInExpression_]

*路径模式*是指向(refer to)常量值或指向没有字段的结构体或没有字段的枚举变体的模式。

非限定路径模式可以指向：

* 枚举变体
* 结构体
* 常量
* 关联常量

限定路径模式只能指向关联常量。

常量不能是联合体类型。结构体常量和枚举常量必须带有 `#[derive(PartialEq, Eq)]` 属性（不只是实现）。

当路径模式指向结构体或枚举变体(枚举只有一个变体)或类型为不可反驳型的常量时，该路径模式是不可反驳型的。当路径模式指向的是可反驳型常量或带有多个变体的枚举时，该路径模式是可反驳型的。

## or 模式

_or模式_是能匹配两个或多个并列子模式（例如：`A | B | C`）中的一个的模式。此模式可以任意嵌套。除了 `let`绑定和函数参数（包括闭包参数）中的模式（此时句法上使用 _PatternNoTopAlt_产生式），or模式在句法上允许在任何其他模式出现的地方出现（这些模式句法上使用 _Pattern_产生式）。

### 静态语义

1. 假定在某个代码深度上给定任意模式 `p` 和 `q`，现假定它们组成模式 `p | q`，则以下情况会导致这种组成的非法：

   + 从 `p` 推断出的类型和从 `q` 推断出的类型不一致，或
   + `p` 和 `q` 引入的绑定标识符不一样，或
   + `p` 和 `q` 中引入的同名绑定标识符的类型和绑定模式中的类型不一致。

   前面提到的所有实例中的类型都必须是精确的，隐式的[类型强转][type coercions]在这里不适用。

2. 当对表达式 `match e_s { a_1 => e_1, ... a_n => e_n }` 做类型检查时，假定在 `e_s` 内部深度为 `d` 的地方存一个表达式片段，那对于此片段，每一个匹配臂 `a_i` 都包含了一个 `p_i | q_i` 来与此段内容进行匹配，但如果表达式片段的类型与 `p_i | q_i` 的类型不一致，则该模式 `p_i | q_i` 被认为是格式错误的。

3. 为了遵从匹配模式的穷尽性检查，模式 `p | q` 被认为同时覆盖了 `p` 和 `q`。对于某些构造器 `c(x, ..)` 来说，此时应用分配律能使 `c(p | q, ..rest)` 与 `c(p, ..rest) | c(q, ..rest)` 覆盖相同的一组匹配值。这个规律可以递归地应用，直到不再有形式为 `p | q`  的嵌套模式。
   
  注意这里的*“构造器”*这个用词，我们并没有特定提到它是元组结构模式，因为它本意是指任何能够生成类型的模式。这包括枚举变量、元组结构、具有命名字段的结构、数组、元组和切片。

### 动态语义

1. 检查对象表达式(scrutinee expression) `e_s` 与深度为 `d` 的模式 `c(p | q, ..rest)`（这里`c`是某种构造器，`p` 和 `q` 是任意的模式，`rest` 是 `c`构造器的任意的可选因子）进行匹配的动态语义与此表达式与 `c(p, ..rest) | c(q, ..rest)` 进行匹配的语法定义相同。

### 无分解符模式的优先级

如本章其他部分所示，有几种类型的模式在语法上没有定义分界符，它们包括标识符模式、引用模式和 or模式。它们组合在一起时，or模式的优先级总是最低的。这允许我们为将来可能的类型特性保留语法空间，同时也可以减少歧义。例如，`x @ A(..) | B(..)` 将导致一个错误，即 `x` 不是在所有模式中都存在绑定关系； `&A(x) | B(x)`将导致不同子模式中的 `x` 之的类型不匹配。


[^译注1]: 请仔细参研[匹配表达式](expressions/match-expr.md)中的 MatchExpression产生式，搞清楚匹配臂(MatchArm)的位置。

[^译注2]: 文字叙述有些晦涩，译者举个例子：假如 `if let &Some(y) = &&&Some(3) {`，此时会首先剥掉等号两边的第一层 `&`号，然后是 `Some(y)` 和 `&&Some(3)`匹配，此时发现是非引用模式和引用匹配上了，就再对 `&&Some(3)` 做重复解引用，解出 `Some(3)`，然后从外部转向内部，见到最后的变量 `y` 和检验对象 `3`，就更新 `y` 的默认绑定方式为 `ref`，所以 `y` 就匹配为 `&3`；如果我们这个例子的变量 `y` 改为 `ref y`，不影响 `y` 的绑定效果；极端的情况 `if let &Some(y) = &&&Some(x) {`，如果 `x` 是可变的，那么此时 `y` 的绑定方式就是 `ref mut`，再进一步极端 `if let &Some(ref y) = &&&Some(x) {`，此时 `y` 的绑定方式仍是 `ref`。

[_GroupedPattern_]: #grouped-patterns
[_IdentifierPattern_]: #identifier-patterns
[_LiteralPattern_]: #literal-patterns
[_MacroInvocation_]: macros.md#macro-invocation
[_ObsoleteRangePattern_]: #range-patterns
[_PathInExpression_]: paths.md#paths-in-expressions
[_PathPattern_]: #path-patterns
[_Pattern_]: #patterns
[_PatternWithoutRange_]: #patterns
[_QualifiedPathInExpression_]: paths.md#qualified-paths
[_RangePattern_]: #range-patterns
[_ReferencePattern_]: #reference-patterns
[_RestPattern_]: #rest-patterns
[_SlicePattern_]: #slice-patterns
[_StructPattern_]: #struct-patterns
[_TuplePattern_]: #tuple-patterns
[_TupleStructPattern_]: #tuple-struct-patterns
[_WildcardPattern_]: #wildcard-pattern

[`Copy`]: special-types-and-traits.md#copy
[IDENTIFIER]: identifiers.md
[enums]: items/enumerations.md
[literals]: expressions/literal-expr.md
[structs]: items/structs.md
[tuples]: types/tuple.md
[scrutinee]: glossary.md#scrutinee
[type coercions]: type-coercions.md
