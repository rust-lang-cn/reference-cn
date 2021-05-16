# 派生

>[derive.md](https://github.com/rust-lang/reference/blob/master/src/attributes/derive.md)\
>commit: a52543267554541a95088b79f46a8bd36f487603 \
>本章译文最后维护日期：2020-11-10

*`derive`属性*允许为数据结构自动生成新的[程序项][items]。它使用 [_MetaListPaths_]元项属性句法（为程序项）指定一系列要实现的 trait 或指定要执行的[派生宏][derive macros]的路径。

例如，下面的派生属性将为结构体 `Foo` 创建一个实现 [`PartialEq`] trait 和 [`Clone`] trait 的[实现(`impl` item)][`impl` item]，类型参数 `T` 将被派生出的实现(`impl`)加上 `PartialEq` 或[^or-and] `Clone` 约束：

```rust
#[derive(PartialEq, Clone)]
struct Foo<T> {
    a: i32,
    b: T,
}
```

上面代码为 `PartialEq` 生成的实现(`impl`)等价于

```rust
# struct Foo<T> { a: i32, b: T }
impl<T: PartialEq> PartialEq for Foo<T> {
    fn eq(&self, other: &Foo<T>) -> bool {
        self.a == other.a && self.b == other.b
    }

    fn ne(&self, other: &Foo<T>) -> bool {
        self.a != other.a || self.b != other.b
    }
}
```

可以通过[过程宏][procedural macros]为自定义的 trait 实现自动派生(`derive`)功能。

## `automatically_derived`属性

*`automatically_derived`属性*会被自动添加到由 `derive`属性为一些内置trait 自动派生的[实现][implementations]中。它对派生出的实现没有直接影响，但是工具和诊断lint 可以使用它来检测这些自动派生的实现。

[^or-and]: 原文后半句是："and the type parameter `T` will be given the `PartialEq` or `Clone` constraints for the appropriate `impl`:"，这里译者也搞不清楚为什么 `PartialEq` 和 `Clone` 之间用了"or"，而不是"and"？这里译者就先采用直译。

[_MetaListPaths_]: ../attributes.md#meta-item-attribute-syntax
[`Clone`]: https://doc.rust-lang.org/std/clone/trait.Clone.html
[`PartialEq`]: https://doc.rust-lang.org/std/cmp/trait.PartialEq.html
[`impl` item]: ../items/implementations.md
[items]: ../items.md
[derive macros]: ../procedural-macros.md#derive-macros
[implementations]: ../items/implementations.md
[items]: ../items.md
[procedural macros]: ../procedural-macros.md#derive-macros

<!-- 2020-11-12-->
<!-- checked -->
