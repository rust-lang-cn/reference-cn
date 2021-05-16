# 术语表

>[glossary.md](https://github.com/rust-lang/reference/blob/master/src/glossary.md)\
>commit: c28dfe483375849678793dbe86c69a1953f3bb00 \
>本章译文最后维护日期：2021-02-21

### 抽象句法树

“抽象句法树”，或“AST”，是编译器编译程序时，程序结构的中间表示形式。\
An ‘abstract syntax tree’, or ‘AST’, is an intermediate representation of the structure of the program when the compiler is compiling it.

### 对齐量

值的对齐量指定值的首选起始存储地址。对齐量总是2的幂次。值的引用必须是对齐的。[更多][alignment]。\
The alignment of a value specifies what addresses values are preferred to start at. Always a power of two. References to a value must be aligned. [More][alignment].

### 元数

元数是指函数或运算符接受的参数个数。例如，`f(2, 3)` 和 `g(4, 6)` 的元数为2，而 `h(8, 2, 6)` 的元数为3。 `!` 运算符的元数为1。\
Arity refers to the number of arguments a function or operator takes. For some examples, `f(2, 3)` and `g(4, 6)` have arity 2, while `h(8, 2, 6)` has arity 3. The `!` operator has arity 1.

### 数组

数组，有时也称为固定大小数组或内联数组，是描述关于元素集合的值，每个元素都可由程序在运行时计算的索引选择。内存模型上，数组占用连续的内存区域。\
An array, sometimes also called a fixed-size array or an inline array, is a value describing a collection of elements, each selected by an index that can be computed at run time by the program. It occupies a contiguous region of memory.

### 关联程序项/关联项

关联程序项是与另一个程序项关联的程序项。关联程序项在 [trait][traits] 中声明，在[实现][implementations]中定义。只有函数、常量和类型别名可以作为关联程序项。它与[自由程序项][free item]形成对比。\
An associated item is an item that is associated with another item. Associated items are defined in [implementations] and declared in [traits]. Only functions, constants, and type aliases can be associated. Contrast to a [free item].

### blanket实现

指为[无覆盖类型](#uncovered-type)实现的任何实现。`impl<T> Foo for T`、`impl<T> Bar<T> for T`、`impl<T> Bar<Vec<T>> for T`、 和 `impl<T> Bar<T> for Vec<T>` 被认为是 blanket实现。但是，`impl<T> Bar<Vec<T>> for Vec<T>` 不被认为是，因为这个 `impl` 中所有的 `T` 的实例都被 `Vec` 覆盖。\
Any implementation where a type appears [uncovered](#uncovered-type). `impl<T> Foo for T`, `impl<T> Bar<T> for T`, `impl<T> Bar<Vec<T>> for T`, and `impl<T> Bar<T> for Vec<T>` are considered blanket impls. However, `impl<T> Bar<Vec<T>> for Vec<T>` is not a blanket impl, as all instances of `T` which appear in this `impl` are covered by `Vec`.

### 约束

约束是对类型或 trait 的限制。例如，如果在函数的形数上设置了约束，则传递给该函数的实参的类型必须遵守该约束。\
Bounds are constraints on a type or trait. For example, if a bound is placed on the argument a function takes, types passed to that function must abide by that constraint.

### 组合算子

组合子是高阶函数，它的参数全是函数或之前定义的组合子。组合子利用这些函数或组合子返回的结果作为入参进行进一步的逻辑计算和输出。组合子可用于以模块化的方式管理控制流。\
Combinators are higher-order functions that apply only functions and earlier defined combinators to provide a result from its arguments. They can be used to manage control flow in a modular fashion.

### 分发

分发是一种机制，用于确定涉及到多态性时实际运行的是哪个版本的代码。分发的两种主要形式是静态分发和动态分发。虽然 Rust 支持静态分发，但它也通过一种称为 trait对象的机制支持动态分发。\
Dispatch is the mechanism to determine which specific version of code is actually run when it involves polymorphism. Two major forms of dispatch are static dispatch and dynamic dispatch. While Rust favors static dispatch, it also supports dynamic dispatch through a mechanism called ‘trait objects’.

### 动态尺寸类型

动态尺寸类型(DST)是一种没有静态已知尺寸或对齐量的类型。\
A dynamically sized type (DST) is a type without a statically known size or alignment.

### 实体

[*实体(entity)*][*entity*]是一种语言结构，在源程序中可以以某种方式被引用，通常是通过[路径(path)][paths]。实体包括[类型][types]、[程序项][items]、[泛型参数][generic parameters]、[变量绑定][variable bindings]、[循环标签][loop labels]、[生存期][lifetimes]、[字段][fields]、[属性][attributes]和[lints]。\
An [*entity*] is a language construct that can be referred to in some way within the source program, usually via a [path][paths]. Entities include [types], [items], [generic parameters], [variable bindings], [loop labels], [lifetimes], [fields], [attributes], and [lints].

### 表达式

表达式是值、常量、变量、运算符/操作符和函数的组合，计算/求值结果为单个值，有或没有副作用都有可能。\
比如，`2 + (3 * 4)` 是一个返回值为14的表达式。\
An expression is a combination of values, constants, variables, operators and functions that evaluate to a single value, with or without side-effects.\
For example, `2 + (3 * 4)` is an expression that returns the value 14.

### 自由程序项

不是任何[实现][implementation][item]的成员的[程序项]，如*自由函数*或*自由常量*。自由程序项是与[关联程序项][associated item]相对的概念。\
An [item] that is not a member of an [implementation], such as a *free function* or a *free const*. Contrast to an [associated item].

### 基础性trait

基础性trait 就是如果为现有的类型实现它，就会为该类型带来突破性改变的 trait。比如 `Fn` 和 `Sized` 就是基础性trait。\
A fundamental trait is one where adding an impl of it for an existing type is a breaking change. The `Fn` traits and `Sized` are fundamental.

### 基本类型构造器

基本类型构造器是这样一种类型，在它之上实现一个 [blanket实现](#blanket-implementation)是一个突破性的改变。`&`、`&mut`、`Box`、和 `Pin` 是基本类型构造器。\
如果任何时候 `T` 都被认为是[本地类型](#local-type)，那 `&T`、`&mut T`、`Box<T>`、和 `Pin<T>` 也被认为是本地类型。基本类型构造器不能[覆盖](#uncovered-type)其他类型。任何时候使用术语“有覆盖类型”时，都默认把`&T`、`&mut T`、`Box<T>`、和`Pin<T>` 排除在外。\
A fundamental type constructor is a type where implementing a [blanket implementation](#blanket-implementation) over it is a breaking change. `&`, `&mut`, `Box`, and `Pin`  are fundamental. \
Any time a type `T` is considered [local](#local-type), `&T`, `&mut T`, `Box<T>`, and `Pin<T>` are also considered local. Fundamental type constructors cannot [cover](#uncovered-type) other types. Any time the term "covered type" is used, the `T` in `&T`, `&mut T`, `Box<T>`, and `Pin<T>` is not considered covered.

### Inhabited

如果类型具有构造函数，因此可以实例化，则该类型是 inhabited。inhabited 类型不是“空的”，因为可以有类型对应的值。与之相对的是 [Uninhabited](#uninhabited)。\
A type is inhabited if it has constructors and therefore can be instantiated. An inhabited type is not "empty" in the sense that there can be values of the type. Opposite of [Uninhabited](#uninhabited).

### 固有实现

单独标称类型上的[实现][implementation] ，注意关键字 `impl` 后直接是标称类型，而非 trait-标称类型对(trait-type pair)上的实现。[更多][inherent implementation]。 \
An [implementation] that applies to a nominal type, not to a trait-type pair. [More][inherent implementation].

### 固有方法

在[固有实现][inherent implementation]中而不是在 trait实现中定义的[方法][method]。\
A [method] defined in an [inherent implementation], not in a trait implementation.

###  初始化

如果一个变量已经被分配了一个值，并且此值还没有被移动走，那此变量就被初始化了。对此变量而言，它会假设它之外的所有其他内存位置都未初始化。只有非安全Rust 可以在不初始化的情况下开辟内存新区域。\
A variable is initialized if it has been assigned a value and hasn't since been moved from. All other memory locations are assumed to be uninitialized. Only unsafe Rust can create such a memory without initializing it.

### 本地 trait

本地 trait 是在当前 crate 中定义的 `trait`。它可以在模块局部定义，也可以是依附于其他类型参数而定义。给定 `trait Foo<T, U>`，`Foo` 总是本地的，不管替代 `T` 和 `U` 的类型是什么。\
A `trait` which was defined in the current crate. A trait definition is local or not independent of applied type arguments. Given `trait Foo<T, U>`, `Foo` is always local, regardless of the types substituted for `T` and `U`.

### Turbofish

表达式中带有泛型参数的路径必须在左尖括号前加上一个 `::`。
这种为表达泛型而结合起来形式（`::<>`）看起来有些像一条鱼。
因此，在口头上就被称为 turbofish句法。\
Paths with generic parameters in expressions must prefix the opening brackets with a `::`.
Combined with the angular brackets for generics, this looks like a fish `::<>`.
As such, this syntax is colloquially referred to as turbofish syntax.

例如：

```rust
let ok_num = Ok::<_, ()>(5);
let vec = [1, 2, 3].iter().map(|n| n * 2).collect::<Vec<_>>();
```

这里必须使用 `::`前缀，以便在逗号分隔的列表中进行多次比较时消除泛型路径可能的歧义。
参见 [the bastion of the turbofish][turbofish test] 中因为没有此前缀的而引起歧义的示例。\
This `::` prefix is required to disambiguate generic paths with multiple comparisons in a comma-separate list.
See [the bastion of the turbofish][turbofish test] for an example where not having the prefix would be ambiguous.

### 本地类型

指在当前 crate 中定义的 `struct`、`enum`、或 `union` 。本地类型不会受到类型参数的影响。`struct Foo` 被认为是本地的，但 `Vec<Foo>` 不是。`LocalType<ForeignType>` 是本地的。类型别名不影响本地性。\
A `struct`, `enum`, or `union` which was defined in the current crate. This is not affected by applied type arguments. `struct Foo` is considered local, but `Vec<Foo>` is not. `LocalType<ForeignType>` is local. Type aliases do not affect locality.

### 名称

[*名称*][*name*]是一个指向[实体](#entity)的[标识符][identifier]或[生存期或循环标签][lifetime or loop label]。*名称绑定(name binding)*是指实体声明时引入了与该实体相关联的标识符或标签。[路径][Paths]、标识符和标签用于引用实体。\
A [*name*] is an [identifier] or [lifetime or loop label] that refers to an [entity](#entity). A *name binding* is when an entity declaration introduces an identifier or label associated with that entity. [Paths], identifiers, and labels are used to refer to an entity.

### 名称解析

[*名称解析*][*Name resolution*]是将[路径][paths]、[标识符][identifiers]、[标签][labels]和[实体(entity)](#entity)声明绑定在一起的的编译过程。\
[*Name resolution*] is the compile-time process of tying [paths], [identifiers], and [labels] to [entity](#entity) declarations.

### 命名空间

*命名空间*是基于名称所引用的[实体](#entity)类型的声明[名称](#name)的逻辑分组。命名空间让在一个命名空间中出现的名称不会与另一个命名空间中的相同名称冲突。\
A *namespace* is a logical grouping of declared [names](#name) based on the kind of [entity](#entity) the name refers to. Namespaces allow the occurrence of a name in one namespace to not conflict with the same name in another namespace.\
在命名空间中，名称统一组织在一个层次结构中，层次结构的每一层都有自己的命名实体集合。\
Within a namespace, names are organized in a hierarchy, where each level of the hierarchy has its own collection of named entities.

### 标称类型

可用路径直接引用的类型。具体来说就是[枚举(`enum`)][enums]、[结构体(`struct`)][structs]、[联合体(`union`)][unions]和 [trait对象][trait objects]。\
Types that can be referred to by a path directly. Specifically [enums], [structs], [unions], and [trait objects].

### 对象安全 trait

可以用作 [trait对象]的 [trait][Traits]。只有遵循特定[规则][object safety]的 trait 才是对象安全的。\
[Traits] that can be used as [trait objects]. Only traits that follow specific [rules][object safety] are object safe.

### 路径

[*路径*][*path*]是一个或多个路径段组成的序列，用于引用当前作用域或某[命名空间](#namespace)层次结构中的[实体](#entity)。\
A [*path*] is a sequence of one or more path segments used to refer to an [entity](#entity) in the current scope or other levels of a [namespace](#namespace) hierarchy.

### 预加载模块集/预导入包

预加载模块集，或者 Rust 预加载模块集，是一个会被导入到每个 crate 中的每个模块的小型程序项集合（其中大部分是 trait）。trait 在预加载模块集中很普遍。\
Prelude, or The Rust Prelude, is a small collection of items - mostly traits - that are imported into every module of every crate. The traits in the prelude are pervasive.

### 作用域

[*作用域*][*scope*]是源文本的一个区域，在该区域中可以直接使用其名称来引用在其下命名的命名[实体](#entity)。\
A [*scope*] is the region of source text where a named [entity](#entity) may be referenced with that name.

### 检验对象\检验对象表达式

检验对象是在匹配(`match`)表达式和类似的模式匹配结构上匹配的表达式。例如，在 `match x { A => 1, B => 2 }` 中，表达式 `x` 是 scrutinee。\
A scrutinee is the expression that is matched on in `match` expressions and similar pattern matching constructs. For example, in `match x { A => 1, B => 2 }`, the expression `x` is the scrutinee.

### 类型尺寸/尺寸

值的尺寸有两个定义。\
第一个是必须分配多少内存来存储这个值。\
第二个是它是在具有该项类型的数组中连续元素之间的字节偏移量。\
它是对齐量的整数倍数，包括零倍。尺寸会根据编译器版本(进行新的优化时)和目标平台(类似于 `usize` 在不同平台上的变化)而变化。\
查看[更多][alignment].
The size of a value has two definitions.\
The first is that it is how much memory must be allocated to store that value.\
The second is that it is the offset in bytes between successive elements in an array with that item type.
It is a multiple of the alignment, including zero. The size can change depending on compiler version (as new optimizations are made) and target platform (similar to how `usize` varies per-platform).\

### 切片

切片是一段连续的内存序列上的具有动态尺寸视图功能的类型，写为 `[T]`。\
它经常以借用的形式出现，可变借用和共享借用都有可能。共享借用切片类型是 `&[T]`，可变借用切片类型是 `&mut [T]`，其中 `T` 表示元素类型。\
A slice is dynamically-sized view into a contiguous sequence, written as `[T]`.\
It is often seen in its borrowed forms, either mutable or shared. The shared slice type is `&[T]`, while the mutable slice type is `&mut [T]`, where `T` represents the element type.

### 语句

语句是编程语言中最小的独立元素，它命令计算机执行一个动作。\
A statement is the smallest standalone element of a programming language that commands a computer to perform an action.

### 字符串字面量

字符串字面量是直接存储在最终二进制文件中的字符串，因此在 `'static` 生存期内是有效的。\
它的类型是借用形式的生存期 `'static` 的字符串切片，即：`&'static str`。\
A string literal is a string stored directly in the final binary, and so will be valid for the `'static` duration.\
Its type is `'static` duration borrowed string slice, `&'static str`.

### 字符串切片(`str`)

字符串切片是 Rust 中最基础的字符串类型，写为 `str`。它经常以借用的形式出现，可变借用和共享借用都有可能。共享借用的字符串切片类型是  `&str`，可变借用的字符串切片类型是 `&mut str`。\
字符串切片总是有效的 UTF-8。\
A string slice is the most primitive string type in Rust, written as `str`. It is often seen in its borrowed forms, either mutable or shared. The shared string slice type is `&str`, while the mutable string slice type is `&mut str`.\
Strings slices are always valid UTF-8.

### Trait

trait 是一种程序项，用于描述类型必须提供的功能。它允许类型对其行为做出某些承诺。\
泛型函数和泛型结构体可以使用 trait 来限制或约束它们所接受的类型。\
A trait is a language item that is used for describing the functionalities a type must provide. It allows a type to make certain promises about its behavior.\
Generic functions and generic structs can use traits to constrain, or bound, the types they accept.

### 无覆盖类型

不作为其他类型的参数出现的类型。例如，`T` 就是无覆盖的，但 `Vec<T>` 中的 `T` 就是有覆盖的。这（种说法）只与类型参数相关。\
A type which does not appear as an argument to another type. For example, `T` is uncovered, but the `T` in `Vec<T>` is covered. This is only relevant for type arguments.

### 未定义行为

非指定的编译时或运行时行为。这可能导致，但不限于：进程终止或崩溃；不正确的、不正确的或非预定计算；或特定于平台的结果。查看[更多][undefined-behavior]\
Compile-time or run-time behavior that is not specified. This may result in, but is not limited to: process termination or corruption; improper, incorrect, or unintended computation; or platform-specific results. [More][undefined-behavior].

### Uninhabited

如果类型没有构造函数，因此永远不能实例化，则该类型是 Uninhabited。一个 Uninhabited 类型是“空的”，意思是该类型没有值。Uninhabited 类型的典型例子是 [never type] `!`，或不带变体的 `enum Never { }`。与之相对的是 [Inhabited](#inhabited)。\
A type is uninhabited if it has no constructors and therefore can never be instantiated. An uninhabited type is "empty" in the sense that there are no values of the type. The canonical example of an uninhabited type is the [never type] `!`, or an enum with no variants `enum Never { }`. Opposite of [Inhabited](#inhabited).

[alignment]: type-layout.md#size-and-alignment
[associated item]: #associated-item
[attributes]: attributes.md
[*entity*]: names.md
[enums]: items/enumerations.md
[fields]: expressions/field-expr.md
[free item]: #free-item
[generic parameters]: items/generics.md
[identifier]: identifiers.md
[identifiers]: identifiers.md
[implementation]: items/implementations.md
[implementations]: items/implementations.md
[inherent implementation]: items/implementations.md#inherent-implementations
[item]: items.md
[items]: items.md
[labels]: tokens.md#lifetimes-and-loop-labels
[lifetime or loop label]: tokens.md#lifetimes-and-loop-labels
[lifetimes]: tokens.md#lifetimes-and-loop-labels
[lints]: attributes/diagnostics.md#lint-check-attributes
[loop labels]: tokens.md#lifetimes-and-loop-labels
[method]: items/associated-items.md#methods
[*Name resolution*]: names/name-resolution.md
[*name*]: names.md
[*namespace*]: names/namespaces.md
[never type]: types/never.md
[object safety]: items/traits.md#object-safety
[*path*]: paths.md
[Paths]: paths.md
[*scope*]: names/scopes.md
[structs]: items/structs.md
[trait objects]: types/trait-object.md
[traits]: items/traits.md
[turbofish test]: https://github.com/rust-lang/rust/blob/master/src/test/ui/bastion-of-the-turbofish.rs
[types]: types.md
[undefined-behavior]: behavior-considered-undefined.md
[unions]: items/unions.md
[variable bindings]: patterns.md

<!-- 2021-1-24-->
<!-- checked -->
