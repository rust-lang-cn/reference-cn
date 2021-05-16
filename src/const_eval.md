# 常量求值

>[const_eval.md](https://github.com/rust-lang/reference/blob/master/src/const_eval.md)\
>commit:  8425f5bad3ac40e807e3f75f13b989944da28b62 \
>本章译文最后维护日期：2021-4-5

常量求值是在编译过程中计算[表达式][[expressions]]结果的过程。（不是所有表达式都可以在编译时求值，也就是说）只有全部表达式的某个子集可以在编译时求值。

## 常量表达式

某些形式的表达式（被称为常量表达式）可以在编译时求值。在[常量(const)上下文](#const-context)中，常量表达式是唯一允许的表达式，并且总是在编译时求值。在其他地方，比如 [let语句][let statements]，常量表达式*可以*在编译时求值，但不能保证总能在此时求值。如果值必须在编译时求得（例如在常量上下文中），则像[数组索引][array indexing]越界或[溢出][overflow]这样的行为都是编译错误。如果不是必须在编译时求值，则这些行为在编译时只是警告，但它们在运行时可能会触发 panic。

下列表达式中，只要它们的所有操作数都是常量表达式，并且求值/计算不会引起任何 [`Drop::drop`][destructors]函数的运行，那这些表达式就是常量表达式。

* [字面量][Literals]。
* [常量参数][Const parameters]。
* 指向[函数项][functions]和[常量项][constants]的[路径][Paths]。不允许递归地定义常量项。
* 指向[静态项][statics]的路径。这种路径只允许出现在静态项的初始化器中。
* [元组表达式][Tuple expressions]。
* [数组表达式][Array expressions]。
* [结构体][Struct]表达式。
* [块表达式][Block expressions]，包括非安全(`unsafe`)块。
    * [let语句][let statements]以及类似这样的不可反驳型[模式][patterns]绑定，包括可变绑定。
    * [赋值表达式][assignment expressions]
    * [复合赋值表达式][compound assignment expressions]
    * [表达式语句][expression statements]
* [字段][Field]表达式。
* 索引表达式，长度为 `usize` 的[数组索引][array indexing]或[切片][slice]。
* [区间表达式][Range expressions]。
* 未从环境捕获变量的[闭包][Closure expressions]。
* 在整型、浮点型、布尔型(`bool`)和字符型(`char`)上做的各种内置运算，包括：[取反][negation]、[算术][arithmetic]、[逻辑][logical]、[比较][comparison] 或 [惰性布尔][lazy boolean]运算。
* 排除借用类型为[内部可变借用][interior mutability]的共享[借用][borrow]表达式。
* 排除解引用裸指针的[解引用操作][dereference operator]。8425f5bad3ac40e807e3f75f13b989944da28b62
  * 指针到地址的强制转换，
  * 函数指针到地址的强制转换，和
  * 到 trait对象的非固定尺寸类型强换(unsizing casts)。
* 调用[常量函数][const functions]和常量方法。
* [loop], [while] 和 [`while let`] 表达式。
* [if], [`if let`] 和 [匹配(match)] 表达式。

## 常量上下文

下述位置是*常量上下文*：

* [数组类型的长度表达式][Array type length expressions]
* [分号分隔的数组创建形式中的长度表达式][array expressions]
* 下述表达式的初始化器(initializer)：
  * [常量项][constants]
  * [静态项][statics]
  * [枚举判别值][enum discriminants]
* [常量型泛型实参][const generic argument]

## 常量函数

*常量函数(const fn)*可以在常量上下文中调用。给一个函数加一个常量(`const`)标志对该函数的任何现有的使用都没有影响，它只限制参数和返回可以使用的类型，并防止在这两个位置上使用不被允许的表达式类型。程序员可以自由地用常量函数去做任何用常规函数能做的事情。

当从常量上下文中调用这类函数时，编译器会在编译时解释该函数。这种解释发生在编译目标环境中，而不是在当前主机环境中。因此，如果是针对一个 `32` 位目标系统进行编译，那么 `usize` 就是 `32` 位，这与在一个 `64` 位还是在一个 `32` 位主机环境中进行编译动作无关。

常量函数有各种限制以确保其可以在编译时可被求值。因此，例如，不可以将随机数生成器编写为常量函数。在编译时调用常量函数将始终产生与运行时调用它相同的结果，即使多次调用也是如此。这个规则有一个例外：如果在极端情况下执行复杂的浮点运算，那么可能得到（非常轻微）不同的结果。建议不要让数组长度和枚举判别值/式依赖于浮点计算。

常量上下文有，但常量函数不具备的显著特性有：

* 浮点运算
  * （在常量函数中，）处理浮点值就像处理只有 `Copy` trait约束的泛型参数一样，不能用它们做任何事，只能复制/移动它们。
* trait对象(`dyn Trait`)/动态分发类型
* 泛型参数上除 `Sized` 之外的泛型约束
* 比较裸指针
* 访问联合体字段
* 调用 [`transmute`]。

相反，以下情况在常量函数中是有可能的，但在常量上下文中则不可能：

* 使用泛型类型和生存期参数。
  * 常量上下文允许有限地使用[常量型泛型形参][const generic parameters]。

[arithmetic]:           expressions/operator-expr.md#arithmetic-and-logical-binary-operators
[array expressions]:    expressions/array-expr.md
[array indexing]:       expressions/array-expr.md#array-and-slice-indexing-expressions
[array indexing]:       expressions/array-expr.md#array-and-slice-indexing-expressions
[array type length expressions]: types/array.md
[assignment expressions]: expressions/operator-expr.md#assignment-expressions
[compound assignment expressions]: expressions/operator-expr.md#compound-assignment-expressions
[block expressions]:    expressions/block-expr.md
[borrow]:               expressions/operator-expr.md#borrow-operators
[cast]:                 expressions/operator-expr.md#type-cast-expressions
[closure expressions]:  expressions/closure-expr.md
[comparison]:           expressions/operator-expr.md#comparison-operators
[const functions]:      items/functions.md#const-functions
[const generic argument]: items/generics.md#const-generics
[const generic parameters]: items/generics.md#const-generics
[constants]:            items/constant-items.md
[Const parameters]:     items/generics.md
[dereference operator]: expressions/operator-expr.md#the-dereference-operator
[destructors]:          destructors.md
[enum discriminants]:   items/enumerations.md#custom-discriminant-values-for-fieldless-enumerations
[expression statements]: statements.md#expression-statements
[expressions]:          expressions.md
[field]:                expressions/field-expr.md
[functions]:            items/functions.md
[grouped]:              expressions/grouped-expr.md
[interior mutability]:  interior-mutability.md
[if]:                   expressions/if-expr.md#if-expressions
[`if let`]:             expressions/if-expr.md#if-let-expressions
[lazy boolean]:         expressions/operator-expr.md#lazy-boolean-operators
[let statements]:       statements.md#let-statements
[literals]:             expressions/literal-expr.md
[logical]:              expressions/operator-expr.md#arithmetic-and-logical-binary-operators
[loop]:                 expressions/loop-expr.md#infinite-loops
[match]:                expressions/match-expr.md
[negation]:             expressions/operator-expr.md#negation-operators
[overflow]:             expressions/operator-expr.md#overflow
[paths]:                expressions/path-expr.md
[patterns]:             patterns.md
[range expressions]:    expressions/range-expr.md
[slice]:                types/slice.md
[statics]:              items/static-items.md
[struct]:               expressions/struct-expr.md
[tuple expressions]:    expressions/tuple-expr.md
[`transmute`]:          https://doc.rust-lang.org/std/mem/fn.transmute.html
[while]:                expressions/loop-expr.md#predicate-loops
[`while let`]:          expressions/loop-expr.md#predicate-pattern-loops
