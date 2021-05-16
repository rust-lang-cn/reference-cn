# 数组和数组索引表达式

>[array-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/array-expr.md)\
>commit: 31dc83fe187a87af2b162801d50f4bed171fecdb \
>本章译文最后维护日期：2021-4-5

## 数组表达式

> **<sup>句法</sup>**\
> _ArrayExpression_ :\
> &nbsp;&nbsp; `[` [_InnerAttribute_]<sup>\*</sup> _ArrayElements_<sup>?</sup> `]`
>
> _ArrayElements_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_Expression_] ( `,` [_Expression_] )<sup>\*</sup> `,`<sup>?</sup>\
> &nbsp;&nbsp; | [_Expression_] `;` [_Expression_]

*数组表达式*用来构建[数组][array]。
数组表达式有两种形式。

第一种形式是在数组中列举出所有的元素值。
这种形式的句法规则通过在方括号中放置统一类型的、逗号分隔的表达式来表现。
这样编写将生成一个包含这些表达式的值的数组，其中数组元素的顺序就是这些表达式写入的时顺序。

第二种形式的句法规则通过在方括号内放置两用个用分号(`;`)分隔的表达式来表现。
分号(`;`)前的表达式被称为*重复体操作数(repeat operand)*
分号(`;`)后的表达式被称为*数组长度操作数(length operand)*
其中，数组长度操作数必须是 `usize` 类型的，并且必须是[常量表达式][constant expression]，比如是[字面量][literal]或[常量项][constant item]。
也就是说，`[a; b]` 这种形式会创建包含 `b` 个 `a` 值的数组。
如果数组长度操作数的值大于 1，则要求 `a` 的类型实现了 [`Copy`][`Copy`]，或 `a` 自己是一个常量项的[路径][path]。

当 `[a; b]` 形式的重复体操作数 `a` 是一个常量项时，其将被计算求值数组长度操作数 `b` 次。
如果数组长度操作数 `b` 为 0，则常量项根本不会被求值。
对于非常量项的表达式，只计算求值一次，然后将结果复制数组长度操作数 `b` 次。

<div class="warning">

警告：当数组长度操作数为 0 时，而重复体操作数是一个非常量项的情况下，目前在 `rustc` 中存在一个 bug，即值 `a` 会被求值，但不会被销毁而导致的内存泄漏。参见 [issue#74836](https://github.com/rust-lang/rust/issues/74836)。

</div>


```rust
[1, 2, 3, 4];
["a", "b", "c", "d"];
[0; 128];              // 内含128个0的数组
[0u8, 0u8, 0u8, 0u8,];
[[1, 0, 0], [0, 1, 0], [0, 0, 1]]; // 二维数组
const EMPTY: Vec<i32> = Vec::new();
[EMPTY; 2];
```

### 数组表达式上的属性

在允许[块表达式上的属性][Inner attributes]存在的那几种表达式上下文中，可以在数组表达式的左括号后直接使用[内部属性][attributes on block expressions]。

## 数组和切片索引表达式

> **<sup>句法</sup>**\
> _IndexExpression_ :\
> &nbsp;&nbsp; [_Expression_] `[` [_Expression_] `]`

[数组][Array]和[切片][slice]类型的值(slice-typed values)可以通过后跟一个由方括号封闭一个类型为 `usize` 的表达式（索引）的方式来对此数组或切片进行索引检索。
如果数组是可变的，则其检索出的[内存位置][memory location]还可以被赋值。

对于数组和切片类型之外的索引表达式 `a[b]` 其实相当于执行 `*std::ops::Index::index(&a, b)`，或者在可变位置表达式上下文中相当于执行 `*std::ops::IndexMut::index_mut(&mut a, b)`。
与普通方法一样，Rust 也将在 `a` 上反复插入解引用操作，直到查找到对上述方法的实现。

数组和切片的索引是从零开始的。数组访问是一个[常量表达式][constant expression]，因此数组索引的越界检查可以在编译时通过检查常量索引值本身进行。
否则，越界检查将在运行时执行，如果此时越界检查未通过，那将把当前线程置于 *panicked 状态*。

```rust,should_panic
// 默认情况下,`unconditional_panic` lint检查会执行 deny 级别的设置，
// 即 crate 在默认情况下会有外部属性设置 `#[deny(unconditional_panic)]`
// 而像 `(["a", "b"])[n]` 这样的简单动态索引检索会被该 lint 检查出来，而提前报错，导致程序被拒绝编译。
// 因此这里调低 `unconditional_panic` 的 lint 级别以通过编译。
#![warn(unconditional_panic)]

([1, 2, 3, 4])[2];        // 3

let b = [[1, 0, 0], [0, 1, 0], [0, 0, 1]];
b[1][2];                  // 多维数组索引

let x = (["a", "b"])[10]; // 告警：索引越界

let n = 10; 
// 译者注：上行可以在 `#![warn(unconditional_panic)]` 被注释的情况下换成
// let n = if true {10} else {0};
// 试试，那下行就不会被 unconditional_panic lint 检查到了
  
let y = (["a", "b"])[n];  // panic

let arr = ["a", "b"];
arr[10];                  // 告警：索引越界
```

数组和切片以外的类型可以通过实现 [Index] trait 和 [IndexMut] trait 来达成数组索引表达式的效果。

[`Copy`]: ../special-types-and-traits.md#copy
[IndexMut]: ../../std/ops/trait.IndexMut.html
[Index]: ../../std/ops/trait.Index.html
[Inner attributes]: ../attributes.md
[_Expression_]: ../expressions.md
[_InnerAttribute_]: ../attributes.md
[array]: ../types/array.md
[attributes on block expressions]: block-expr.md#attributes-on-block-expressions
[constant expression]: ../const_eval.md#constant-expressions
[constant item]: ../items/constant-items.md
[literal]: ../tokens.md#literals
[memory location]: ../expressions.md#place-expressions-and-value-expressions
[path]: path-expr.md
[slice]: ../types/slice.md
