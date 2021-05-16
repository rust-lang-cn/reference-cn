# 命名空间

>[use-declarations.md](https://github.com/rust-lang/reference/blob/master/src/names/namespaces.md)\
>commit: 10da84befd0e79f5e1490912a02cb74368cd3f4a \
>本章译文最后维护日期：2021-1-25

*命名空间*是已声明的[名称][names]的逻辑分组。根据名称所指的实体类型，名称被分隔到不同的命名空间中。
名称空间允许一个名称空间中出现的名称与另一个名称空间中的相同，且不会导致冲突。

在命名空间中，名称被组织在不同的层次结构中，层次结构的每一层都有自己的命名实体集合。

程序有几个不同的命名空间，每个名称空间包含不同种类的实体。使用名称时将根据上下文来在不同的命名空间中去查找该名称的声明，[名称解析][name resolution]一章有讲到这些。

下面是一系列命名空间及其对应实体的列表：

* 类型命名空间
    * [模块声明][Module declarations]
    * [外部crate声明][External crate declarations]
    * [外部预导入包][External crate prelude]中的程序项
    * [结构体][Struct]声明、[联合体][union]声明、[枚举][enum]声明和枚举变体声明
    * [trait项声明][Trait item declarations]
    * [类型别名][Type aliases]
    * [关联类型声明][Associated type declarations]
    * 内置类型：[布尔型][boolean]、[数字型][numeric]和[文本型][textual]
    * [泛型类型参数][Generic type parameters]
    * [`Self`类型][`Self` type]
    * [工具类属性模块][Tool attribute modules]
* 值命名空间
    * [函数声明][Function declarations]
    * [常量项声明][Constant item declarations]
    * [静态项声明][Static item declarations]
    * [结构体构造器][Struct constructors]
    * [枚举变体构造器][Enum variant constructors]
    * [`Self`构造器][`Self` constructors]
    * [泛型常量参数][Generic const parameters]
    * [关联常量声明][Associated const declarations]
    * [关联函数声明][Associated function declarations]
    * 本地绑定 — [`let`], [`if let`], [`while let`], [`for`], [`match`]臂, [函数参数][function parameters], [闭包参数][closure parameters]
    * [闭包][closure]捕获的变量
* 宏命名空间
    * [`macro_rules`声明][`macro_rules` declarations]
    * [内置属性][Built-in attributes]
    * [工具类属性][Tool attributes]
    * [类函数过程宏][Function-like procedural macros]
    * [派生宏][Derive macros]
    * [辅助派生宏][Derive macro helpers]
    * [属性宏][Attribute macros]
* 生存期命名空间
    * [泛型生存期参数][Generic lifetime parameters]
* 标签命名空间[^rustc-lifetime-shadow]
    * [循环标签][Loop labels]

如何清晰地使用不同命名空间中的同名名称的示例：

```rust
// Foo 在类型命名空间中引入了一个类型，在值命名空间中引入了一个构造函数
struct Foo(u32);

// 宏`Foo`在宏命名空间中声明
macro_rules! Foo {
    () => {};
}

// 参数`f` 的类型中的 `Foo` 指向类型命名空间中的 `Foo`
// `'Foo` 引入一个生存期命名空间里的新的生存期
fn example<'Foo>(f: Foo) {
    // `Foo` 引用值命名空间里的 `Foo`构造器。
    let ctor = Foo;
    // `Foo` 引用宏命名空间里的 `Foo`宏。
    Foo!{}
    // `'Foo` 引入一个标签命名空间里的标签。
    'Foo: loop {
        // `'Foo` 引用 `'Foo`生存期参数, `Foo` 引用类型命名空间中的类型。
        let x: &'Foo Foo;
        // `'Foo` 引用了 `'Foo`标签.
        break 'Foo;
    }
}
```

## 无命名空间的命名实体

下面的实体有显式的名称，但是这些名称不属于任何特定的命名空间。

### 字段

即使结构体、枚举和联合体的字段被命名，但这些命名字段并不存在于任何显式的命名空间中。它们只能通过[字段表达式][field expression]访问，该表达式只检测被访问的特定类型的字段名。

### use声明

[use声明][use declaration]命名了导入到当前作用域中的实体，但 `use`项本身不属于任何特定的命名空间。相反，它可以在多个名称空间中引入别名，这取决于所导入的程序项类型。

<!-- TODO: describe how `use` works on the use-declarations page, and link to it here. -->

[^rustc-lifetime-shadow]: 目前，在同一个作用域中，标签和生命周期使用相同名称时，`rustc` 会警告出现命名重复，但编译器内部仍然会独立地区别对待它们。这是一个关于该语言可能扩展的未来兼容性警告。具体见[PR #24162](https://github.com/rust-lang/rust/pull/24162)。

[`for`]: ../expressions/loop-expr.md#iterator-loops
[`if let`]: ../expressions/if-expr.md#if-let-expressions
[`let`]: ../statements.md#let-statements
[`macro_rules` declarations]: ../macros-by-example.md
[`match`]: ../expressions/match-expr.md
[`Self` constructors]: ../paths.md#self-1
[`Self` type]: ../paths.md#self-1
[`while let`]: ../expressions/loop-expr.md#predicate-pattern-loops
[Associated const declarations]: ../items/associated-items.md#associated-constants
[Associated function declarations]: ../items/associated-items.md#associated-functions-and-methods
[Associated type declarations]: ../items/associated-items.md#associated-types
[Attribute macros]: ../procedural-macros.md#attribute-macros
[boolean]: ../types/boolean.md
[Built-in attributes]: ../attributes.md#built-in-attributes-index
[closure parameters]: ../expressions/closure-expr.md
[closure]: ../expressions/closure-expr.md
[Constant item declarations]: ../items/constant-items.md
[Derive macro helpers]: ../procedural-macros.md#derive-macro-helper-attributes
[Derive macros]: ../procedural-macros.md#derive-macros
[entity]: ../glossary.md#entity
[Enum variant constructors]: ../items/enumerations.md
[enum]: ../items/enumerations.md
[External crate declarations]: ../items/extern-crates.md
[External crate prelude]: preludes.md#extern-prelude
[field expression]: ../expressions/field-expr.md
[Function declarations]: ../items/functions.md
[function parameters]: ../items/functions.md#function-parameters
[Function-like procedural macros]: ../procedural-macros.md#function-like-procedural-macros
[Generic const parameters]: ../items/generics.md#const-generics
[Generic lifetime parameters]: ../items/generics.md
[Generic type parameters]: ../items/generics.md
[Loop labels]: ../expressions/loop-expr.md#loop-labels
[Module declarations]: ../items/modules.md
[name resolution]: name-resolution.md
[names]: ../names.md
[numeric]: ../types/numeric.md
[Static item declarations]: ../items/static-items.md
[Struct constructors]: ../items/structs.md
[Struct]: ../items/structs.md
[textual]: ../types/textual.md
[Tool attribute modules]: ../attributes.md#tool-attributes
[Tool attributes]: ../attributes.md#tool-attributes
[Trait item declarations]: ../items/traits.md
[Type aliases]: ../items/type-aliases.md
[union]: ../items/unions.md
[use declaration]: ../items/use-declarations.md
