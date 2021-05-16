# 循环

>[loop-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/loop-expr.md)\
>commit: d23f9da8469617e6c81121d9fd123443df70595d \
>本章译文最后维护日期：2021-5-6

> **<sup>句法</sup>**\
> _LoopExpression_ :\
> &nbsp;&nbsp; [_LoopLabel_]<sup>?</sup> (\
> &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; [_InfiniteLoopExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_PredicateLoopExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_PredicatePatternLoopExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_IteratorLoopExpression_]\
> &nbsp;&nbsp; )

[_LoopLabel_]: #loop-labels
[_InfiniteLoopExpression_]: #infinite-loops
[_PredicateLoopExpression_]: #predicate-loops
[_PredicatePatternLoopExpression_]: #predicate-pattern-loops
[_IteratorLoopExpression_]: #iterator-loops

Rust支持四种循环表达式：

*   [`loop`表达式](#infinite-loops)表示一个无限循环。
*   [`while`表达式](#predicate-loops)不断循环，直到谓词为假。
*   [`while let`表达式](#predicate-pattern-loops)循环测试给定模式。
*   [`for`表达式](#iterator-loops)从迭代器中循环取值，直到迭代器为空。

所有四种类型的循环都支持 [`break`表达式](#break-expressions)、[`continue`表达式](#continue-expressions)和[循环标签(label)](#loop-labels)。
只有 `loop`循环支持对循环体[非平凡求值(evaluation to non-trivial values)](#break-and-loop-values)[^译注1]。

## 无限循环

> **<sup>句法</sup>**\
> _InfiniteLoopExpression_ :\
> &nbsp;&nbsp; `loop` [_BlockExpression_]

`loop`表达式会不断地重复地执行它代码体内的代码：`loop { println!("I live."); }`。

没有包含关联的 `break`表达式的 `loop`表达式是发散的，并且具有类型 [`!`]。
包含相应 `break`表达式的 `loop`表达式可以结束循环，并且此表达式的类型必须与 `break`表达式的类型兼容。

## 谓词循环

> **<sup>句法</sup>**\
> _PredicateLoopExpression_ :\
> &nbsp;&nbsp; `while` [_Expression_]<sub>_排除结构体表达式_</sub> [_BlockExpression_]

`while`循环从对[布尔型][boolean]的循环条件操作数求值开始。
如果循环条件操作数的求值结果为 `true`，则执行循环体块，然后控制流返回到循环条件操作数。如果循环条件操作数的求值结果为 `false`，则 `while`表达式完成。

举个例子：

```rust
let mut i = 0;

while i < 10 {
    println!("hello");
    i = i + 1;
}
```

## 谓词模式循环

> **<sup>句法</sup>**\
> [_PredicatePatternLoopExpression_] :\
> &nbsp;&nbsp; `while` `let` [_Pattern_] `=` [_Expression_]<sub>_排除结构体表达式和惰性布尔运算符表达式_</sub>
>              [_BlockExpression_]

`while let`循环在语义上类似于 `while`循环，但它用 `let`关键字后紧跟着一个模式、一个 `=`、一个[检验对象(scrutinee)][scrutinee]表达式和一个块表达式，来替代原来的条件表达式。
如果检验对象表达式的值与模式匹配，则执行循环体块，然后控制流再返回到模式匹配语句。如果不匹配，则 `while`表达式执行完成。

```rust
let mut x = vec![1, 2, 3];

while let Some(y) = x.pop() {
    println!("y = {}", y);
}

while let _ = 5 {
    println!("不可反驳模式总是会匹配成功");
    break;
}
```

`while let`循环等价于包含匹配(`match`)表达式的 `loop`表达式。
如下：

<!-- ignore: expansion example -->
```rust,ignore
'label: while let PATS = EXPR {
    /* loop body */
}
```

等价于

<!-- ignore: expansion example -->
```rust,ignore
'label: loop {
    match EXPR {
        PATS => { /* loop body */ },
        _ => break,
    }
}
```

可以使用操作符 `|` 指定多个模式。
这与匹配(`match`)表达式中的 `|` 具有相同的语义：

```rust
let mut vals = vec![2, 3, 1, 2, 2];
while let Some(v @ 1) | Some(v @ 2) = vals.pop() {
    // 打印 2, 2, 然后 1
    println!("{}", v);
}
```

与 [`if let`表达式][`if let` expressions]的情况一样，检验表达式不能是一个[懒惰布尔运算符表达式][_LazyBooleanOperatorExpression_]。

## 迭代器循环

> **<sup>句法</sup>**\
> _IteratorLoopExpression_ :\
> &nbsp;&nbsp; `for` [_Pattern_] `in` [_Expression_]<sub>_排除结构体表达式_</sub>
>              [_BlockExpression_]

`for`表达式是一个用于在 `std::iter::IntoIterator` 的某个迭代器实现提供的元素上进行循环的语法结构。
如果迭代器生成一个值，该值将与此 `for`表达式提供的不可反驳型模式进行匹配，执行循环体，然后控制流返回到 `for`循环的头部。
如果迭代器为空了，则 `for`表达式执行完成。

`for`循环遍历数组内容的示例：

```rust
let v = &["apples", "cake", "coffee"];

for text in v {
    println!("I like {}.", text);
}
```

`for`循环遍历一个整数序列的例子：

```rust
let mut sum = 0;
for n in 1..11 {
    sum += n;
}
assert_eq!(sum, 55);
```

`for`循环等价于后面的块表达式。

<!-- ignore: expansion example -->
```rust,ignore
'label: for PATTERN in iter_expr {
    /* loop body */
}
```

等价于：

<!-- ignore: expansion example -->
```rust,ignore
{
    let result = match IntoIterator::into_iter(iter_expr) {
        mut iter => 'label: loop {
            let mut next;
            match Iterator::next(&mut iter) {
                Option::Some(val) => next = val,
                Option::None => break,
            };
            let PATTERN = next;
            let () = { /* loop body */ };
        },
    };
    result
}
```

这里的 `IntoIterator`、`Iterator` 和 `Option` 是标准库的程序项(standard library item)，不是当前作用域中解析的的任何名称。
变量名 `next`、`iter` 和 `val` 也仅用于表述需要，实际上它们不是用户可以输入的名称。

> **注意**：上面代码里使用外层 `matche` 来确保 `iter_expr` 中的任何[临时值][temporary values]在循环结束前不会被销毁。
> `next` 先声明后赋值是因为这样能让编译器更准确地推断出类型。

## 循环标签

> **<sup>句法</sup>**\
> _LoopLabel_ :\
> &nbsp;&nbsp; [LIFETIME_OR_LABEL] `:`

一个循环表达式可以选择设置一个*标签*。
这类标签被标记为循环表达式之前的生存期（标签），如 `'foo: loop { break 'foo; }`、`'bar: while false {}`、`'humbug: for _ in 0..0 {}`。
如果循环存在标签，则嵌套在该循环中的带此标签的 `break`表达式和 `continue`表达式可以退出此标签标记的循环层或将控制流返回至此标签标记的循环层的头部。
具体请参见后面的 [break表达式](#break-expressions)和 [continue表达式](#continue-expressions)。

## `break`表达式

> **<sup>句法</sup>**\
> _BreakExpression_ :\
> &nbsp;&nbsp; `break` [LIFETIME_OR_LABEL]<sup>?</sup> [_Expression_]<sup>?</sup>

当遇到 `break` 时，相关的循环体的执行将立即结束，例如：

```rust
let mut last = 0;
for x in 1..100 {
    if x > 12 {
        break;
    }
    last = x;
}
assert_eq!(last, 12);
```

`break`表达式通常与包含 `break`表达式的最内层 `loop`、`for`或 `while`循环相关联，但是可以使用[循环标签](#loop-labels)来指定受影响的循环层（此循环层必须是封闭该 break表达式的循环之一）。
例如：

```rust
'outer: loop {
    while true {
        break 'outer;
    }
}
```

`break`表达式只允许在循环体内使用，它有 `break`、`break 'label` 或（[参见后面](#break-and-loop-values)）`break EXPR` 或 `break 'label EXPR` 这四种形式。

## `continue`表达式

> **<sup>句法</sup>**\
> _ContinueExpression_ :\
> &nbsp;&nbsp; `continue` [LIFETIME_OR_LABEL]<sup>?</sup>

当遇到 `continue` 时，相关的循环体的当前迭代将立即结束，并将控制流返回到循环头。
在 `while`循环的情况下，循环头是控制循环的条件表达式。
在 `for`循环的情况下，循环头是控制循环的调用表达式。

与 `break` 一样，`continue` 通常与最内层的循环相关联，但可以使用 `continue 'label` 来指定受影响的循环层。
`continue`表达式只允许在循环体内部使用。

## `break`和`loop`返回值

当使用 `loop`循环时，可以使用 `break`表达式从循环中返回一个值，通过形如 `break EXPR` 或 `break 'label EXPR` 来返回，其中 `EXPR` 是一个表达式，它的结果被从 `loop`循环中返回。
例如：

```rust
let (mut a, mut b) = (1, 1);
let result = loop {
    if b > 10 {
        break b;
    }
    let c = a + b;
    a = b;
    b = c;
};
// 斐波那契数列中第一个大于10的值：
assert_eq!(result, 13);
```

如果 `loop` 有关联的 `break`，则不认为该循环是发散的，并且 `loop`表达式的类型必须与每个 `break`表达式的类型兼容。
其后不跟表达式的 `break` 被认为与后跟 `()` 的`break`表达式的效果相同。

[^译注1]: 求得 `()` 类型以外的值。  

[`!`]: ../types/never.md
<!-- 上面这几个链接从原文来替换时需小心 -->
[LIFETIME_OR_LABEL]: ../tokens.md#lifetimes-and-loop-labels
[_BlockExpression_]: block-expr.md
[_Expression_]: ../expressions.md
[_Pattern_]: ../patterns.md
[`match` expression]: match-expr.md
[boolean]: ../types/boolean.md
[scrutinee]: ../glossary.md#scrutinee
[temporary values]: ../expressions.md#temporaries
[_LazyBooleanOperatorExpression_]: operator-expr.md#lazy-boolean-operators
[`if let` expressions]: if-expr.md#if-let-expressions
