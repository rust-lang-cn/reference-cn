# 结构体表达式

>[struct-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/struct-expr.md)\
>commit: eb5290329316e96c48c032075f7dbfa56990702b \
>本章译文最后维护日期：2021-02-21

> **<sup>句法</sup>**\
> _StructExpression_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _StructExprStruct_\
> &nbsp;&nbsp; | _StructExprTuple_\
> &nbsp;&nbsp; | _StructExprUnit_
>
> _StructExprStruct_ :\
> &nbsp;&nbsp; [_PathInExpression_] `{` [_InnerAttribute_]<sup>\*</sup> (_StructExprFields_ | _StructBase_)<sup>?</sup> `}`
>
> _StructExprFields_ :\
> &nbsp;&nbsp; _StructExprField_ (`,` _StructExprField_)<sup>\*</sup> (`,` _StructBase_ | `,`<sup>?</sup>)
>
> _StructExprField_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [IDENTIFIER]\
> &nbsp;&nbsp; | ([IDENTIFIER] | [TUPLE_INDEX]) `:` [_Expression_]
>
> _StructBase_ :\
> &nbsp;&nbsp; `..` [_Expression_]
>
> _StructExprTuple_ :\
> &nbsp;&nbsp; [_PathInExpression_] `(`\
> &nbsp;&nbsp; &nbsp;&nbsp; [_InnerAttribute_]<sup>\*</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp; ( [_Expression_] (`,` [_Expression_])<sup>\*</sup> `,`<sup>?</sup> )<sup>?</sup>\
> &nbsp;&nbsp; `)`
>
> _StructExprUnit_ : [_PathInExpression_]

*结构体表达式*创建结构体或联合体的值。它由指向[结构体][struct]程序项、[枚举变体][enum variant]、[联合体][union]程序项的路径，以及与此程序项的字段对应的值组成。
结构体表达式有三种形式：结构体(struct)、元组结构体(tuple)和单元结构体(unit)。

下面是结构体表达式的示例：

```rust
# struct Point { x: f64, y: f64 }
# struct NothingInMe { }
# struct TuplePoint(f64, f64);
# mod game { pub struct User<'a> { pub name: &'a str, pub age: u32, pub score: usize } }
# struct Cookie; fn some_fn<T>(t: T) {}
Point {x: 10.0, y: 20.0};
NothingInMe {};
TuplePoint(10.0, 20.0);
TuplePoint { 0: 10.0, 1: 20.0 }; // 效果和上一行一样
let u = game::User {name: "Joe", age: 35, score: 100_000};
some_fn::<Cookie>(Cookie);
```

## 结构体表达式的字段设置

用花括号把字段括起来的结构体表达式允许以任意顺序指定每个字段的值。字段名与值之间用冒号分隔。

[联合体][union]类型的值只能使用此句法创建，并且只能指定一个字段。

## 函数式更新句法

结构体表达式构建一个结构体类型的值时可以以 `..` 后跟一个表达式的句法结尾，这种句法表示这是一种函数式更新(functional update)。
`..` 后跟的表达式（此表达式被称为此函数式更新的基(base)）必须与正在构造的新结构体值是同一种结构体类型的。

整个结构体表达式先为已指定的字段使用已给定的值，然后再从基表达式(base expression)里为剩余未指定的字段移动或复制值。
与所有结构体表达式一样，此结构体类型的所有字段必须是[可见的][visible]，甚至那些没有显式命名的字段也是如此。

```rust
# struct Point3d { x: i32, y: i32, z: i32 }
let mut base = Point3d {x: 1, y: 2, z: 3};
let y_ref = &mut base.y;
Point3d {y: 0, z: 10, .. base}; // OK, 只有 base.x 获取进来了
drop(y_ref);
```

带花括号的结构体表达式不能直接用在[循环][loop]表达式或 [if]表达式的头部，也不能直接用在 [if let]或[匹配][match]表达式的[检验对象(scrutinee)][scrutinee]上。
但是，如果结构体表达式在另一个表达式内（例如在[圆括号][parentheses]内），则可以用在这些情况下。

构造元组结构体时其字段名可以是代表索引的十进制整数数值。
这中表达方法还可以与基结构体一起使用来填充其余未指定的索引：

```rust
struct Color(u8, u8, u8);
let c1 = Color(0, 0, 0);  // 创建元组结构体的典型方法。
let c2 = Color{0: 255, 1: 127, 2: 0};  // 按索引来指定字段。
let c3 = Color{1: 0, ..c2};  // 使用基的字段值来填写结构体的所有其他字段。
```

### 初始化结构体字段的快捷方法

当使用字段的名字（注意不是位置索引数字）初始化某数据结构（结构体、枚举、联合体）时，允许将 `fieldname: fieldname` 写成 `fieldname` 这样的简化形式。
这种句法让代码更少重复，更加紧凑。
例如：

For example:
For example:
```rust
# struct Point3d { x: i32, y: i32, z: i32 }
# let x = 0;
# let y_value = 0;
# let z = 0;
Point3d { x: x, y: y_value, z: z };
Point3d { x, y: y_value, z };
```

## 元组结构体表达式

用圆括号括起字段的结构体表达式构造出来的结构体为元组结构体。
虽然为了完整起见，也把它作为一个特定的（结构体）表达式列在这里，但实际上它等价于执行元组结构体构造器的[调用表达式][call expression]。例如：

```rust
struct Position(i32, i32, i32);
Position(0, 0, 0);  // 创建元组结构体的典型方法。
let c = Position;  // `c` 是一个接收3个参数的函数。
let pos = c(8, 6, 7);  // 创建一个 `Position` 值。
```

## 单元结构体表达式

单元结构体表达式只是单元结构体程序项(unit struct item)的路径。
也是指向此单元结构体的值的隐式常量。
单元结构体的值也可以用无字段结构体表达式来构造。例如：

```rust
struct Gamma;
let a = Gamma;  // Gamma的值。
let b = Gamma{};  // 和`a`的值完全一样。
```

## 结构体表达式上的属性

在允许[块表达式上的属性][Inner attributes]存在的那几种表达式上下文中，可以在结构体表达式的左括号后直接使用[内部属性][attributes on block expressions]。

[IDENTIFIER]: ../identifiers.md
[Inner attributes]: ../attributes.md
[TUPLE_INDEX]: ../tokens.md#tuple-index
[_Expression_]: ../expressions.md
[_InnerAttribute_]: ../attributes.md
[_PathInExpression_]: ../paths.md#paths-in-expressions
[attributes on block expressions]: block-expr.md#attributes-on-block-expressions
[call expression]: call-expr.md
[enum variant]: ../items/enumerations.md
[if let]: if-expr.md#if-let-expressions
[if]: if-expr.md#if-expressions
[loop]: loop-expr.md
[match]: match-expr.md
[parentheses]: grouped-expr.md
[struct]: ../items/structs.md
[union]: ../items/unions.md
[visible]: ../visibility-and-privacy.md
[scrutinee]: ../glossary.md#scrutinee
