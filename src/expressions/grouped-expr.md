# 圆括号表达式(分组表达式)

>[grouped-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/grouped-expr.md)\
>commit: 31dc83fe187a87af2b162801d50f4bed171fecdb \
>本章译文最后维护日期：2021-4-5


> **<sup>句法</sup>**\
> _GroupedExpression_ :\
> &nbsp;&nbsp; `(` [_InnerAttribute_]<sup>\*</sup> [_Expression_] `)`

由圆括号封闭的表达式的求值结果就是在其内的表达式的求值结果。
在表达式内部，圆括号可用于显式地指定表达式内部的求值顺序。

*圆括号表达式(parenthesized expression)*包装单个表达式，并对该表达式求值。
圆括号表达式的句法规则就是一对圆括号封闭一个被称为*封闭操作数(enclosed operand)*的表达式。

圆括号表达式被求值为其封闭操作数的值。
与其他表达式不同，圆括号表达式可以是[位置表达式或值表达式][place]。
当封闭操作数是位置表达式时，它是一个位置表达式；当封闭操作数是一个值表达式是，它是一个值表达式。

圆括号可用于显式修改表达式中的子表达式的优先顺序。

圆括号表达式的一个例子：

```rust
let x: i32 = 2 + 3 * 4;
let y: i32 = (2 + 3) * 4;
assert_eq!(x, 14);
assert_eq!(y, 20);
```

当调用结构体的函数指针类型的成员时，必须使用括号，示例如下：

```rust
# struct A {
#    f: fn() -> &'static str
# }
# impl A {
#    fn f(&self) -> &'static str {
#        "The method f"
#    }
# }
# let a = A{f: || "The field f"};
#
assert_eq!( a.f (), "The method f");
assert_eq!((a.f)(), "The field f");
```

## 分组表达式上的属性

在允许[块表达式上的属性][Inner attributes]存在的那几种表达式上下文中，可以在分组表达式的左括号后直接使用[内部属性][attributes on block expressions]。

[Inner attributes]: ../attributes.md
[_Expression_]: ../expressions.md
[_InnerAttribute_]: ../attributes.md
[attributes on block expressions]: block-expr.md#attributes-on-block-expressions
md#attributes-on-block-expressions
[place]: ../expressions.md#place-expressions-and-value-expressions
