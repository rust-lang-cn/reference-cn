# 元组和元组索引表达式

>[tuple-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/tuple-expr.md)\
>commit: 09142b820fe8713c4cba451713c7d11e67d7fbd8 \
>本章译文最后维护日期：2021-4-6

## 元组表达式

> **<sup>句法</sup>**\
> _TupleExpression_ :\
> &nbsp;&nbsp; `(` [_InnerAttribute_]<sup>\*</sup> _TupleElements_<sup>?</sup> `)`
>
> _TupleElements_ :\
> &nbsp;&nbsp; ( [_Expression_] `,` )<sup>+</sup> [_Expression_]<sup>?</sup>

*元组表达式*用来构建[元组值][tuple type]。

元组表达式的句法规则为：一对圆括号封闭的以逗号分隔的表达式列表，这些表达式被称为*元组初始化操作数(tuple initializer operands)*。
为了避免和[圆括号表达式][parenthetical expression]混淆，一元元组表达式的元组初始化操作数后的逗号不能省略。

元组表达式是一个[值表达式][value expression]，它会被求值计算成一个元组类型的新值。
元组初始化操作数的数量构成元组的元数(arity)。
没有元组初始化操作数的元组表达式生成单元元组(unit tuple)。
对于其他元组表达式，第一个被写入的元组初始化操作数初始化第 0 个元素，随后的操作数依次初始化下一个开始的元素。
例如，在元组表达式 `('a', 'b', 'c')` 中，`'a'` 初始化第 0 个元素的值，`'b'` 初始化第 1 个元素，`'c'` 初始化第2个元素。

元组表达式和相应类型的示例：

| 表达式                | 类型          |
| -------------------- | ------------ |
| `()`                 | `()` (unit)  |
| `(0.0, 4.5)`         | `(f64, f64)` |
| `("x".to_string(), )` | `(String, )`  |
| `("a", 4usize, true)`| `(&'static str, usize, bool)` |

### 元组表达式上的属性

在允许[块表达式上的属性][Inner attributes]存在的那几种表达式上下文中，可以在元组表达式的左括号后直接使用[内部属性][attributes on block expressions]。

## 元组索引表达式

> **<sup>句法</sup>**\
> _TupleIndexingExpression_ :\
> &nbsp;&nbsp; [_Expression_] `.` [TUPLE_INDEX]

*元组索引表达式*被用来存取[元组][tuple type]或[元组结构体][tuple structs]的字段。

元组索引表达式的句法规则为：一个被称为*元组操作数(tuple operand)*的表达式后跟一个 `.`，最后再后跟一个元组索引。
*元组索引*的句法规则要求该索引必须写成一个不能有前导零、下划线和后缀的[十进制字面量][decimal literal]的形式。
例如 `0` 和 `2` 是合法的元祖索引，但 `01`、`0_`、`0i32` 这些不行。

元组操作数的类型必须是[元组类型][tuple type]或[元组结构体][tuple structs]。
元组索引必须是元组操作数类型的字段的名称。（译者注：这句感觉原文表达有问题，这里也给出原文 The tuple index must be a name of a field of the type of the tuple operand.）

对元组索引表达式的求值计算除了能求取其元组操作数的对应位置的值之外没有其他作用。
作为[位置表达式][place expression]，元组索引表达式的求值结果是元组操作数字段的位置，该字段与元组索引同名。

元组索引表达式示例：

```rust
// 索引检索一个元组
let pair = ("a string", 2);
assert_eq!(pair.1, 2);

// 索引检索一个元组结构体
# struct Point(f32, f32);
let point = Point(1.0, 0.0);
assert_eq!(point.0, 1.0);
assert_eq!(point.1, 0.0);
```

> **注意**：与字段访问表达式不同，元组索引表达式可以是[调用表达式][call expression]的函数操作数。
> （这之所以可行，）因为元组索引表达式不会与方法调用相混淆，因为方法名不可能是数字。

> **注意**：虽然数组和切片也有元素，但它们必须使用[数组或切片索引表达式][array or slice indexing expression]或[切片模式][slice pattern]去访问它们的元素。

[_Expression_]: ../expressions.md
[_InnerAttribute_]: ../attributes.md
[array or slice indexing expression]: array-expr.md#array-and-slice-indexing-expressions
[attributes on block expressions]: block-expr.md#attributes-on-block-expressions
[call expression]: ./call-expr.md
[decimal literal]: ../tokens.md#integer-literals
[field access expressions]: ./field-expr.html#field-access-expressions
[inner attributes]: ../attributes.md
[operands]: ../expressions.md
[parenthetical expression]: grouped-expr.md
[place expression]: ../expressions.md#place-expressions-and-value-expressions
[slice pattern]: ../patterns.md#slice-patterns
[tuple type]: ../types/tuple.md
[tuple struct]: ../types/struct.md
[TUPLE_INDEX]: ../tokens.md#tuple-index
[value expression]: ../expressions.md#place-expressions-and-value-expressions
