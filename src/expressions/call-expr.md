# 调用表达式

>[call-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/call-expr.md)\
>commit: 31dc83fe187a87af2b162801d50f4bed171fecdb \
>本章译文最后维护日期：2021-4-5

> **<sup>句法</sup>**\
> _CallExpression_ :\
> &nbsp;&nbsp; [_Expression_] `(` _CallParams_<sup>?</sup> `)`
>
> _CallParams_ :\
> &nbsp;&nbsp; [_Expression_]&nbsp;( `,` [_Expression_] )<sup>\*</sup> `,`<sup>?</sup>

*调用表达式*用来调用函数。
调用表达式的句法规则为：一个被称作*函数操作数(function operand)*的表达式，后跟一个圆括号封闭的逗号分割的被称为*参数操作数(argument operands)*的表达式列表。
如果函数最终返回，则此调用表达式执行完成。
对于[非函数类型][non-function types]，表达式 `f(...)` 会使用 [`std::ops::Fn`]、[`std::ops::FnMut`] 或 [`std::ops::FnOnce`] 这些 trait 上的某一方法，选择使用哪个要看 `f` 如何获取其输入的参数，具体就是看是通过引用、可变引用、还是通过获取所有权来获取的。
如有需要，也可通过自动借用。
Rust 也会根据需要自动对 `f` 作解引用处理。

下面是一些调用表达式的示例：

```rust
# fn add(x: i32, y: i32) -> i32 { 0 }
let three: i32 = add(1i32, 2i32);
let name: &'static str = (|| "Rust")();
```

## 函数调用的消歧

为获得更直观的[完全限定的句法规则][fully-qualified syntax]，Rust 对所有函数调都作了糖化(sugar)处理。
根据当前作用域内的程序项调用的二义性，函数调用有可能需要完全限定。

> **注意**：过去，Rust 社区在文档、议题、RFC 和其他社区文章中使用了术语“确定性函数调用句法(Unambiguous Function Call Syntax)”、“通用函数调用句法(Universal Function Call Syntax)” 或 “UFCS”。
> 但是，这个术语缺乏描述力，可能还会混淆当前的议题。
> 我们在这里提起这个词是为了便于搜索。

少数几种情况下经常会出现一些导致方法调用或关联函数调用的接受者或引用对象不明确的情况。这些情况可包括：

* 作用域内的多个 trait 为同一类型定义了相同名称的方法
* 自动解引(Auto-`deref`)用搞不定的情况；例如，区分智能指针本身的方法和指针所指对象上的方法
* 不带参数的方法，就像 [`default()`] 这样的和返回类型的属性(properties)的，如 [`size_of()`]

为了解决这种二义性，程序员可以使用更具体的路径、类型或 trait 来明确指代他们想要的方法或函数。

例如：

```rust
trait Pretty {
    fn print(&self);
}

trait Ugly {
  fn print(&self);
}

struct Foo;
impl Pretty for Foo {
    fn print(&self) {}
}

struct Bar;
impl Pretty for Bar {
    fn print(&self) {}
}
impl Ugly for Bar {
    fn print(&self) {}
}

fn main() {
    let f = Foo;
    let b = Bar;

    // 我们可以这样做，因为对于`Foo`，我们只有一个名为 `print` 的程序项
    f.print();
    // 对于 `Foo`来说，这样是更明确了，但没必要
    Foo::print(&f);
    // 如果你不喜欢简洁的话，那，也可以这样
    <Foo as Pretty>::print(&f);

    // b.print(); // 错误： 发现多个 `print`
    // Bar::print(&b); // 仍错： 发现多个 `print`

    // 必要，因为作用域内的多个程序项定义了 `print`
    <Bar as Pretty>::print(&b);
}
```

更多细节和动机说明请参考[RFC 132]。

[RFC 132]: https://github.com/rust-lang/rfcs/blob/master/text/0132-ufcs.md
[_Expression_]: ../expressions.md
[`default()`]: https://doc.rust-lang.org/std/default/trait.Default.html#tymethod.default
[`size_of()`]: https://doc.rust-lang.org/std/mem/fn.size_of.html
[`std::ops::FnMut`]: https://doc.rust-lang.org/std/ops/trait.FnMut.html
[`std::ops::FnOnce`]: https://doc.rust-lang.org/std/ops/trait.FnOnce.html
[`std::ops::Fn`]: https://doc.rust-lang.org/std/ops/trait.Fn.html
[automatically dereferenced]: field-expr.md#automatic-dereferencing
[fully-qualified syntax]: ../paths.md#qualified-paths
[non-function types]: ../types/function-item.md
