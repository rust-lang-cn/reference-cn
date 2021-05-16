# 字段访问表达式

>[field-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/field-expr.md)\
>commit: 23672971a16c69ea894bef24992b74912cfe5d25 \
>本章译文最后维护日期：2021-4-5

> **<sup>句法</sup>**\
> _FieldExpression_ :\
> &nbsp;&nbsp; [_Expression_] `.` [IDENTIFIER]

*字段表达式(field expression)*是计算求取[结构体][struct]或[联合体][union]的字段的内存位置的[位置表达式][place expression]。
当操作数[可变][mutable]时，其字段表达式也是可变的。

*字段表达式(field expression)*的句法规则为：一个被称为*容器操作数(container operand)*的表达式后跟一个单点号(`.`)，最后是一个[标识符][identifier]。
字段表达式后面不能再紧跟着一个被圆括号封闭起来的逗号分割的表达式列表（这种表示这是一个[方法调用表达式][method call expression]）。
因此字段表达式不能是调用表达式的函数调用者。

字段表达式代表[结构体(`struct`)][struct]或[联合体(`union`)][union]的字段。要调用存储在结构体的字段中的函数，需要在此字段表达式外加上圆括号。

> **注意**：如果要在调用表达式中使用它(来调用函数)，要把此字段表达式先用圆括号包装成一个[圆括号表达式][parenthesized expression]。
>
> ```rust
> # struct HoldsCallable<F: Fn()> { callable: F }
> let holds_callable = HoldsCallable { callable: || () };
>
> // 非法: 会被解析为调用 "callable"方法
> // holds_callable.callable();
>
> // 合法
> (holds_callable.callable)();
> ```

示例：

<!-- ignore: needs lots of support code -->
```rust,ignore
mystruct.myfield;
foo().x;
(Struct {a: 10, b: 20}).a;
(mystruct.function_field)() // 调用表达式里包含一个字段表达式
```

## 自动解引用

如果容器操作数的类型实现了 [`Deref`] 或 [`DerefMut`][`Deref`]（这取决于该操作数是否为[可变][mutable]），则会尽可能多次地*自动解引用(automatically dereferenced)*，以使字段访问成为可能。
这个过程也被简称为*自动解引用(autoderef)*。

## 借用

当借用时，结构体的各个字段以及对结构体的整体引用都被视为彼此分离的实体。
如果结构体没有实现 [`Drop`]，同时该结构体又存储在局部变量中，（这种各个字段被视为彼此分离的单独实体的逻辑）还适用于每个字段的移出(move out)。
但如果对 [`Box`]化之外的用户自定义类型执行自动解引用，这（种各个字段被视为彼此分离的单独实体的逻辑）就不适用了。

```rust
struct A { f1: String, f2: String, f3: String }
let mut x: A;
# x = A {
#     f1: "f1".to_string(),
#     f2: "f2".to_string(),
#     f3: "f3".to_string()
# };
let a: &mut String = &mut x.f1; // x.f1 被可变借用
let b: &String = &x.f2;         // x.f2 被不可变借用
let c: &String = &x.f2;         // 可以被再次借用
let d: String = x.f3;           // 从 x.f3 中移出
```

[_Expression_]: ../expressions.md
[`Box`]: ../special-types-and-traits.md#boxt
[`Deref`]: ../special-types-and-traits.md#deref-and-derefmut
[`drop`]: ../special-types-and-traits.md#drop
[IDENTIFIER]: ../identifiers.md
[call expression]: call-expr.md
[method call expression]: method-call-expr.md
[mutable]: ../expressions.md#mutability
[parenthesized expression]: grouped-expr.md
[place expression]: ../expressions.md#place-expressions-and-value-expressions
[struct]: ../items/structs.md
[union]: ../items/unions.md
