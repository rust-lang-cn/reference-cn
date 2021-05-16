# 闭包表达式

>[closure-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/closure-expr.md)\
>commit: d23f9da8469617e6c81121d9fd123443df70595d \
>本章译文最后维护日期：2021-5-6

> **<sup>句法</sup>**\
> _ClosureExpression_ :\
> &nbsp;&nbsp; `move`<sup>?</sup>\
> &nbsp;&nbsp; ( `||` | `|` _ClosureParameters_<sup>?</sup> `|` )\
> &nbsp;&nbsp; ([_Expression_] | `->` [_TypeNoBounds_]&nbsp;[_BlockExpression_])
>
> _ClosureParameters_ :\
> &nbsp;&nbsp; _ClosureParam_ (`,` _ClosureParam_)<sup>\*</sup> `,`<sup>?</sup>
>
> _ClosureParam_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> [_PatternNoTopAlt_]&nbsp;( `:` [_Type_] )<sup>?</sup>

*闭包表达式*，也被称为 lambda表达式或 lambda，它定义了一个[闭包类型][closure type]，并把此表达式求值计算为该类型的值。
闭包表达式的句法规则为：先是一个可选的 `move`关键字，后跟一对管道定界符(`|`)封闭的逗号分割的被称为*闭包参数(closure parameters)*的[模式][patterns]列表（每个闭包参数都可选地通过 `:` 后跟其类型），再可选地通过 `->` 后跟一个*返回类型*，最后是被称为*闭包体操作数(closure body operand)* 的表达式。
代表闭包参数的每个模式后面的可选类型是该模式的类型标注(type annotations)。
如果存在返回类型，则闭包体表达式必须是一个普通的[块][block](表达式)。

闭包表达式本质是将一组参数映射到参数后面的表达式的函数。
与 [`let`绑定][`let` binding]一样，闭包参数也是不可反驳型[模式][patterns]的，其类型标注是可选的，如果没有给出，则从上下文推断。
每个闭包表达式都有一个唯一的匿名类型。

特别值得注意的是闭包表达式能*捕获它们被定义时的环境中的变量(capture their environment)*，而普通的[函数定义][function definitions]则不能。
如果没有关键字 `move`，闭包表达式将[推断它该如何从其环境中捕获每个变量][infers how it captures each variable from its environment]，它倾向于通过共享引用来捕获，从而有效地借用闭包体中用到的所有外部变量。
如果有必要，编译器会推断出应该采用可变引用，还是应该从环境中移动或复制值（取决于这些变量的类型）。
闭包可以通过前缀关键字 `move` 来强制通过复制值或移动值的方式捕获其环境变量。
这通常是为了确保当前闭包的生存期类型为 `'static`。

编译器将通过闭包对其捕获的变量的处置方式来确定此闭包类型将实现的[闭包trait][closure traits]。
如果所有捕获的类型都实现了 [`Send`] 和/或 [`Sync`]，那么此闭包类型也实现了 `Send` 和/或 `Sync`。
这些存在这些 trait，函数可以通过泛型的方式接受各种闭包，即便闭包的类型名无法被确切指定。


## 闭包 trait 的实现

当前闭包类型实现哪一个闭包trait 依赖于该闭包如何捕获变量和这些变量的类型。
了解闭包如何和何时实现 `Fn`、`FnMut` 和 `FnOnce` 这三类 trait，请参看[调用trait 和自动强转][call traits and coercions]那一章。
如果所有捕获的类型都实现了 [`Send`] 和/或 [`Sync`]，那么此闭包类型也实现了 `Send` 和/或 `Sync`。

## Example
## 示例

在下面例子中，我们定义了一个名为 `ten_times` 的函数，它接受高阶函数参数，然后我们传给它一个闭包表达式作为实参并调用它。
之后又定义了一个使用移动语义从环境中捕获变量的闭包表达式来供该函数调用。

```rust
fn ten_times<F>(f: F) where F: Fn(i32) {
    for index in 0..10 {
        f(index);
    }
}

ten_times(|j| println!("hello, {}", j));
// 带类型标注 i32
ten_times(|j: i32| -> () { println!("hello, {}", j) });

let word = "konnichiwa".to_owned();
ten_times(move |j| println!("{}, {}", word, j));
```

## 闭包参数上的属性

闭包参数上的属性遵循与[常规函数参数][regular function parameters]上相同的规则和限制。

[_Expression_]: ../expressions.md
[_BlockExpression_]: block-expr.md
[_TypeNoBounds_]: ../types.md#type-expressions
[_PatternNoTopAlt_]: ../patterns.md
[_Type_]: ../types.md#type-expressions
[`let` binding]: ../statements.md#let-statements
[`Send`]: ../special-types-and-traits.md#send
[`Sync`]: ../special-types-and-traits.md#sync
[_OuterAttribute_]: ../attributes.md
[block]: block-expr.md
[call traits and coercions]: ../types/closure.md#call-traits-and-coercions
[closure type]: ../types/closure.md
[function definitions]: ../items/functions.md
[patterns]: ../patterns.md
[regular function parameters]: ../items/functions.md#attributes-on-function-parameters
