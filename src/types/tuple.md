# 元组类型

>[tuple.md](https://github.com/rust-lang/reference/blob/master/src/types/tuple.md)\
>commit: ff6c3dc120ce6d06548b9045c43103f58720ee62 \
>本章译文最后维护日期：2021-4-6

> **<sup>句法</sup>**\
> _TupleType_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `(` `)`\
> &nbsp;&nbsp; | `(` ( [_Type_] `,` )<sup>+</sup> [_Type_]<sup>?</sup> `)`

*元组类型*是由其他类型的异构列表组合成的一类结构化类型[^1]。

元组类型的语法规则为一对圆括号封闭的逗号分割的类型列表。
为和[圆括号类型][parenthesized type]区分开来，一元元组的元素类型后面需要有一个逗号。

元组类型的字段数量等同于其封闭的异构类型列表的长度。
字段的数量决定元组的*元数(arity)*。
有 `n` 个字段的元组叫做 *n元元组(n-ary tuple)*。
例如，有两个字段的元组就是二元元组。

元组的字段用它在列表中的位置数字来索引。
第一个字段索引为 `0`。
第二个字段索引为 `1`。
然后以此类推。
每个字段的类型都是元组类型列表中相同位置的类型。

出于方便和历史原因，不带元素(`()`)的元组类型通常被称为*单元(unit)*或*单元类型(unit type)*。
它的值也被称为*单元*或*单元值*。

元组类型的示例：

* `()` (单元)
* `(f64, f64)`
* `(String, i32)`
* `(i32, String)` (跟前一个示例类型不一样)
* `(i32, f64, Vec<String>, Option<bool>)`

这种类型的值是使用[元组表达式][tuple expression]来构造的。
此外，如果没有其他有意义的值可供求得/返回，很多种表达式都将生成单元值。
元组字段可以通过[元组索引表达式][tuple index expression]或[模式匹配][pattern matching]来访问。

[^1]: 结构化类型的特点就是其内部对等位置的类型如果是相等的，那么这些结构化类型就是相等的。有关元组的标称类型版本，请参见[元组结构体][tuple structs]。

[_Type_]: ../types.md#type-expressions
[parenthesized type]: ../types.md#parenthesized-types
[pattern matching]: ../patterns.md#tuple-patterns
[tuple expression]: ../expressions/tuple-expr.md#tuple-expressions
[tuple index expression]: ../expressions/tuple-expr.md#tuple-indexing-expressions
[tuple structs]: ./struct.md

<!-- 2021-1-10-->
<!-- checked -->
