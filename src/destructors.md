# 析构函数

>[destructors.md](https://github.com/rust-lang/reference/blob/master/src/destructors.md)\
>commit: b2d11240bd9a3a6dd34419d0b0ba74617b23d77e \
>本章译文最后维护日期：2020-11-16


当一个[初始化][initialized]了的[变量][variable]或[临时变量][temporary]超出[作用域](#drop-scopes)时，其*析构函数(destructor)*将运行，或者说它将被*销毁(dropped)*。此外[赋值][Assignment]操作也会运行其左操作数的析构函数（如果它已经初始化了）。如果变量已部分初始化了，则只销毁其已初始化的字段。

类型 `T` 的析构函数由以下内容组成：

1. 如果有约束 `T: Drop`, 则调用 [`<T as std::ops::Drop>::drop`]
2. 递归运行其所有字段的析构函数。
    * [结构体(`struct`)][struct]的字段按照声明顺序被销毁。
    * 活动[枚举变体][enum variant]的字段按声明顺序销毁。
    * [元组][tuple]中的字段按顺序销毁。
    * [数组][array]或拥有所有权的[切片][slice]的元素的销毁顺序是从第一个元素到最后一个元素。
    * [闭包][closure]通过移动(move)语义捕获的变量的销毁顺序未明确指定。
    * [trait对象][Trait objects]的销毁会运行其非具名基类(underlying type)的析构函数。
    * 其他类型不会导致任何进一步的销毁动作发生。

如果析构函数必须手动运行，比如在实现自定义的智能指针时，可以使用标准库函数 [`std::ptr::drop_in_place`]。

举些（析构函数的）例子：

```rust
struct PrintOnDrop(&'static str);

impl Drop for PrintOnDrop {
    fn drop(&mut self) {
        println!("{}", self.0);
    }
}

let mut overwritten = PrintOnDrop("当覆写时执行销毁");
overwritten = PrintOnDrop("当作用域结束时执行销毁");

let tuple = (PrintOnDrop("Tuple first"), PrintOnDrop("Tuple second"));

let moved;
// 没有析构函数在赋值时运行
moved = PrintOnDrop("Drops when moved");
// 这里执行销毁，但随后变量进入未初始化状态
moved;

// 未初始化不会被销毁
let uninitialized: PrintOnDrop;

// 在部分移动之后，后续销毁动作只销毁剩余字段。
let mut partial_move = (PrintOnDrop("first"), PrintOnDrop("forgotten"));
// 执行部分移出，只留下 `partial_move.0` 处于初始化状态
core::mem::forget(partial_move.1);
// 当 partial_move 的作用域结束时, 这里就只有第一个字段被销毁。
```

## 销毁作用域

每个变量或临时变量都与一个*销毁作用域(drop scope)*[^译注1]相关联。当控制流离开一个销毁作用域时，与该作用域关联的所有变量都将按照其声明（对变量而言）或创建（对临时变量而言）的相反顺序销毁。

销毁作用域是在将 [`for`]、[`if let`] 和 [`while let`] 这些表达式替换为等效的 [`match`]表达式之后确定的。在确定销毁作用域这事儿上，对重载操作符与内置操作符不做区分[^译注2]，匹配模式的变量[绑定方式(binding modes)][binding modes]也不去考虑。

给定一个函数或闭包，存在以下的销毁作用域：

* 整个函数
* 每个[语句][statement]
* 每个[表达式][expression]
* 每个块，包括函数体
    * 当在[块表达式][block expression]上时，整个块和整个块表达式的销毁作用域是相同的
* 匹配(`match`)表达式的每条匹配臂(arm)

销毁作用域相互嵌套如规则下。当同时离开多个作用域时，比如从函数返回时，变量会从内层的向层依次销毁。

* 整个函数作用域是最外层的作用域。
* 函数体块包含在整个函数作用域内。
* 表达式语句中的表达式的父作用域是该语句的作用域。
* [`let`语句][`let` statement]的初始化器(initializer)的父作用域是 `let`语句的作用域。
* 语句作用域的父作用域是包含该语句的块作用域。
* 匹配守卫(`match` guard)表达式的父作用域是该守卫所在的匹配臂的作用域。
* 在匹配表达式(`match` expression)里的 `=>` 之后的表达式的父作用域是此表达式对应的匹配臂所在的那个作用域。
* 匹配臂的作用域的父作用域是它所在的匹配表达式(`match` expression)的作用域。
* 所有其他作用域的父作用域都是直接封闭该表达式的作用域。

### 函数参数的作用域

所有函数参数都在整个函数体的作用域内有效，因此在对函数求值时，它们是最后被销毁的。实参会在其内部值被形参的模式绑定之后销毁。

```rust
# struct PrintOnDrop(&'static str);
# impl Drop for PrintOnDrop {
#     fn drop(&mut self) {
#         println!("drop({})", self.0);
#     }
# }
// 先销毁第二个参数, 接下来是 `y`, 然后是第一个参数r, 最后是 `x`
fn patterns_in_parameters(
    (x, _): (PrintOnDrop, PrintOnDrop),
    (_, y): (PrintOnDrop, PrintOnDrop),
) {}

// 销毁顺序是 3 2 0 1
patterns_in_parameters(
    (PrintOnDrop("0"), PrintOnDrop("1")),
    (PrintOnDrop("2"), PrintOnDrop("3")),
);
```

### 本地变量的作用域

在 `let`语句中声明的局部变量与包含此 `let`语句的块作用域相关联。在匹配(`match`)表达式中声明的局部变量与声明它们的匹配(`match`)臂的匹配臂作用域相关联。

```rust
# struct PrintOnDrop(&'static str);
# impl Drop for PrintOnDrop {
#     fn drop(&mut self) {
#         println!("drop({})", self.0);
#     }
# }
let declared_first = PrintOnDrop("在外层作用域内最后销毁");
{
    let declared_in_block = PrintOnDrop("在内层作用域内销毁");
}
let declared_last = PrintOnDrop("在外层作用域内最先销毁");
```

如果在一个匹配(`match`)表达式的同一个匹配臂中使用了多个模式，则毁顺序不确定。（译者注：这里译者不确定后半句翻译是否准确，这里给出原文：then an unspecified pattern will be used to determine the drop order.）

### 临时作用域

表达式的*临时作用域*用于该表达在位置上下文中求出的结果被保存进的那个临时变量的作用域。有些情况下，此表达式求出的结果会被[提升][promoted]，则此表达式不存在临时作用域。[^译注3]

除了[生存期扩展](#temporary-lifetime-extension)之外，表达式的临时作用域是包含该表达式的最小作用域，它适用于以下情况之一:

* 整个函数体。
* 一条语句。
* [`if`]表达式、[`while`]表达式 或 [`loop`]表达式这三种表达式的代码体。
* `if`表达式的 `else`块。
* `if`表达式的条件表达式，`while`表达式的条件表达式，或匹配表达式中的匹配(`match`)守卫。
* 匹配臂上的表达式。
* [惰性布尔表达式][lazy boolean expression]的第二操作数。

> **注意**：
>
> 在函数体的最终表达式(final expression)中创建的临时变量会在任何具名变量销毁*之后*销毁，因为这里没有更小的封闭它的临时作用域。
>
> 匹配表达式的[检验对象][scrutinee]表达式本身不是一个临时作用域（，但它内部可以包含临时作用域），因此可以在匹配(`match`)表达式之后销毁检验对象表达式中的临时作用域。例如，`match 1 { ref mut z => z };` 中的 `1` 所在的临时变量一直存活到此语句结束。

一些示例：

```rust
# struct PrintOnDrop(&'static str);
# impl Drop for PrintOnDrop {
#     fn drop(&mut self) {
#         println!("drop({})", self.0);
#     }
# }
let local_var = PrintOnDrop("local var");

// 在条件表达式执行后立即销毁
if PrintOnDrop("If condition").0 == "If condition" {
    // 在此块的末尾处销毁
    PrintOnDrop("If body").0
} else {
    unreachable!()
};

// 在此条语句的末尾处销毁
(PrintOnDrop("first operand").0 == ""
// 在 ) 处销毁
|| PrintOnDrop("second operand").0 == "")
// 在此表达式的末尾处销毁
|| PrintOnDrop("third operand").0 == "";

// 在函数末尾处，局部变量之后销毁之后销毁
// 将下面这段更改为一个包含返回(return)表达式的语句将使临时变量在本地变量之前被删除。
// 如果把此临时变量绑定到一个变量，然后返回这个变量，也会先删除这个临时变量
match PrintOnDrop("Matched value in final expression") {
    // 在条件表达式执行后立即销毁
    _ if PrintOnDrop("guard condition").0 == "" => (),
    _ => (),
}
```

### 操作数

在同一表达式中，在对其他操作数求值时，也会创建临时变量来将已求值的操作数的结果保存起来。临时变量与该操作数所属的表达式的作用域相关联。因为一旦表达式求值，临时变量就被移走了，所以销毁它们没有任何效果和意义，除非整个表达式的某一操作数出现异常，导致表达式求值失败，或提前返回，或触发了 panic。

```rust
# struct PrintOnDrop(&'static str);
# impl Drop for PrintOnDrop {
#     fn drop(&mut self) {
#         println!("drop({})", self.0);
#     }
# }
loop {
    // 元组表达式未结束求值就提前返回了，所以其操作数按声明的反序销毁
    (
        PrintOnDrop("Outer tuple first"),
        PrintOnDrop("Outer tuple second"),
        (
            PrintOnDrop("Inner tuple first"),
            PrintOnDrop("Inner tuple second"),
            break,
        ),
        PrintOnDrop("Never created"),
    );
}
```

### 常量提升

如果可以通过借用的方式将某一值表达式写入常量，并且还能通过解引用此借用的方式来解出原始写入的表达式，并且如果这种做法也不更改运行时行为，那 Rust 会将值表达式提升到静态(`'static`) slot 作用域内。也就是说，提升的表达式可以在编译时求值，这求得的值不具备[内部可变性][interior mutability]或不包含[析构函数][destructors]（。这些属性是根据可能的值来确定的，例如 `&None` 的类型总是 `&'static Option<_>`，因为 `&None` 的值是唯一确定的）。

### 临时生存期扩展

> **注意**：临时生存期扩展的确切规则可能还会改变。这里只描述了当前的行为表现。

`let`语句中表达式的临时作用域有时会*扩展*到包含此 `let`语句的块作用域内。根据某些句法规则，当通常的临时作用域太小时，就会这样做。例如：

```rust
let x = &mut 0;
// 通常上面存储0的临时变量（或者临时位置）到这里就会被丢弃，但这里是一直存在到块的末尾。
println!("{}", x);
```

如果一个借用、解引用、字段或元组索引表达式有一个扩展的临时作用域，那么它们的操作数也会同样扩展。如果索引表达式有扩展的临时作用域，那么被索引的表达式也会一并扩展。

#### 基于模式的扩展

*扩展临时作用域的模式(extending pattern)*是下面任一：

* 绑定方式为引用或可变引用的[标识符模式][identifier pattern]。
* [结构体(`struct`)模式][struct pattern]、[元组模式][tuple pattern]、[元组结构体模式][tuple struct pattern]或[切片模式][slice pattern]，其中它们至少有一个直接子模式是扩展临时作用域的模式。

所以 `ref x`、`V(ref x)` 和 `[ref x, y]` 都是扩展临时作用域的模式，但是  `x`、`&ref x` 和 `&(ref x,)` 不是。

如果 `let`语句中的模式是扩展临时作用域的模式，那么初始化器表达式中的临时作用域将被扩展。

#### 基于表达式的扩展

对于带有初始化器的 let语句来说，*扩展临时作用域的表达式(extending expression)*是以下表达式之一：

* 初始化表达式(initializer expression)。
* 扩展临时作用域的[借用表达式][borrow expression]的操作数。
* 扩展临时作用域的[数组][array expression]、[强制转换(cast)][cast expression]、[花括号括起来的结构体][struct expression]或[元组][tuple expression]表达式的操作数。
* 任何扩展临时作用域的[块表达式][block expression]的最终表达式(final expression);

因此，在 `&mut 0`、`(&1, &mut 2)` 和 `Some { 0: &mut 3 }` 中的借用表达式都是扩展临时作用域的表达式。在 `&0 + &1` 和一些 `Some(&mut 0)` 中的借用不是：它们在句法上是函数调用表达式。

任何扩展了临时作用域的借用表达式的操作数的临时作用域都随此表达式的临时作用域的扩展而扩展。

#### 示例

这是一些带有扩展的临时作用域的表达式：

```rust
# fn temp() {}
# trait Use { fn use_temp(&self) -> &Self { self } }
# impl Use for () {}
// 在这些情况下，存储 `temp()` 结果的临时变量与x在同一个作用域中。
let x = &temp();
let x = &temp() as &dyn Send;
let x = (&*&temp(),);
let x = { [Some { 0: &temp(), }] };
let ref x = temp();
let ref x = *&temp();
# x;
```

下面是一些表达式没有扩展临时作用域的例子：

```rust,compile_fail
# fn temp() {}
# trait Use { fn use_temp(&self) -> &Self { self } }
# impl Use for () {}
// 在这些情况下，存储 `temp()` 结果的临时变量只存活到 let语句结束。

let x = Some(&temp());         // ERROR
let x = (&temp()).use_temp();  // ERROR
# x;
```

## 阻断运行析构函数

在 Rust 中，即便类型不是 `'static`，禁止运行析构函数也是允许的，也是安全的。[`std::mem::ManuallyDrop`] 提供了一个包装器(wrapper)来防止变量或字段被自动销毁。

[^译注1]: 后文有时也直接简称为作用域。

[^译注2]: 这里说这句是因为操作符的操作数也涉及到销毁作用域范围的确定。

[^译注3]: 对这句话，这里译者按自己的理解再翻译一遍：一个表达式在位置表达式上使用时会被求值，如果此时没有具名变量和此值绑定，那就会先被保存进一个临时变量里，临时作用域就是伴随这此临时变量而生成。此作用域通常在此表达式所在的语句结束时结束，但如果求出的值被通过借用绑定给具名变量，此作用域会扩展到此具名变量的作用域（后面[生存期扩展](#temporary-lifetime-extension)会讲到）。如果求出的值比较特殊，这个作用域还会提升到全局作用域，这就是所谓的[常量提升][promoted]。

[Assignment]: expressions/operator-expr.md#assignment-expressions
[binding modes]: patterns.md#binding-modes
[closure]: types/closure.md
[destructors]: destructors.md
[expression]: expressions.md
[identifier pattern]: patterns.md#identifier-patterns
[initialized]: glossary.md#initialized
[interior mutability]: interior-mutability.md
[lazy boolean expression]: expressions/operator-expr.md#lazy-boolean-operators
[place context]: expressions.md#place-expressions-and-value-expressions
[promoted]: destructors.md#constant-promotion
[scrutinee]: glossary.md#scrutinee
[statement]: statements.md
[temporary]: expressions.md#temporaries
[variable]: variables.md

[array]: types/array.md
[enum variant]: types/enum.md
[slice]: types/slice.md
[struct]: types/struct.md
[Trait objects]: types/trait-object.md
[tuple]: types/tuple.md

[slice pattern]: patterns.md#slice-patterns
[struct pattern]: patterns.md#struct-patterns
[tuple pattern]: patterns.md#tuple-patterns
[tuple struct pattern]: patterns.md#tuple-struct-patterns

[array expression]: expressions/array-expr.md#array-expressions
[block expression]: expressions/block-expr.md
[borrow expression]: expressions/operator-expr.md#borrow-operators
[cast expression]: expressions/operator-expr.md#type-cast-expressions
[struct expression]: expressions/struct-expr.md
[tuple expression]: expressions/tuple-expr.md#tuple-expressions

[`for`]: expressions/loop-expr.md#iterator-loops
[`if let`]: expressions/if-expr.md#if-let-expressions
[`if`]: expressions/if-expr.md#if-expressions
[`let` statement]: statements.md#let-statements
[`loop`]: expressions/loop-expr.md#infinite-loops
[`match`]: expressions/match-expr.md
[`while let`]: expressions/loop-expr.md#predicate-pattern-loops
[`while`]: expressions/loop-expr.md#predicate-loops

[`<T as std::ops::Drop>::drop`]: https://doc.rust-lang.org/std/ops/trait.Drop.html#tymethod.drop
[`std::ptr::drop_in_place`]: https://doc.rust-lang.org/std/ptr/fn.drop_in_place.html
[`std::mem::ManuallyDrop`]: https://doc.rust-lang.org/std/mem/struct.ManuallyDrop.html

<!-- 2020-11-12-->
<!-- checked -->
