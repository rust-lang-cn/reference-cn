{{#include types-redirect.html}}
# 类型

>[types.md](https://github.com/rust-lang/reference/blob/master/src/types.md)\
>commit: af1cf6d3ca3b7a8c434c142148742aa912e37c34 \
>本章译文最后维护日期：2020-11-14

Rust 程序中的每个变量、程序项和值都有一个类型。*值*的*类型*定义了该如何解释用于保存它的内存数据，以及定义了可以对该值执行的操作。

内置的类型以非平凡的方式(in nontrivial ways)紧密地集成到语言中，这种方式是不可能在用户定义的类型中模拟的。用户定义的类型功能有限。

内置类型列表：

* 原生类型(primitive types):
    * [布尔型(Boolean)][Boolean] — `true` 或 `false`
    * [数字类(Numeric)][Numeric] — 整型(integer) 和 浮点型(float)
    * [文本类(Textual)][Textual] — 字符型(`char`) 和 字符串切片(`str`)
    * [never类型][Never] — `!` — 没有值的类型
*  序列类型(sequence types)：
    * [元组(Tuple)][Tuple]
    * [数组(Array)][Array]
    * [切片(Slice)][Slice]
* 用户自定义类型(user-defined types)：
    * [结构体(Struct)][Struct]
    * [枚举(Enum)][Enum]
    * [联合体(Union)][Union]
* 函数类型(function types)：
    * [函数(Functions)][Functions]
    * [闭包(Closures)][Closures]
* 指针类型(pointer types)：
    * [引用(References)][References]
    * [裸指针(Raw pointers)][Raw pointers]
    * [函数指针(Function pointers)][Function pointers]
* trait类型(Trait types):
    * [trait对象(Trait objects)][Trait objects]
    * [实现trait(Impl trait)][Impl trait]

## 类型表达式

> **<sup>句法</sup>**\
> _Type_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _TypeNoBounds_\
> &nbsp;&nbsp; | [_ImplTraitType_]\
> &nbsp;&nbsp; | [_TraitObjectType_]
>
> _TypeNoBounds_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_ParenthesizedType_]\
> &nbsp;&nbsp; | [_ImplTraitTypeOneBound_]\
> &nbsp;&nbsp; | [_TraitObjectTypeOneBound_]\
> &nbsp;&nbsp; | [_TypePath_]\
> &nbsp;&nbsp; | [_TupleType_]\
> &nbsp;&nbsp; | [_NeverType_]\
> &nbsp;&nbsp; | [_RawPointerType_]\
> &nbsp;&nbsp; | [_ReferenceType_]\
> &nbsp;&nbsp; | [_ArrayType_]\
> &nbsp;&nbsp; | [_SliceType_]\
> &nbsp;&nbsp; | [_InferredType_]\
> &nbsp;&nbsp; | [_QualifiedPathInType_]\
> &nbsp;&nbsp; | [_BareFunctionType_]\
> &nbsp;&nbsp; | [_MacroInvocation_]

上表中的 _Type_ 文法规则中定义的各种*类型表达式*都是某个具体类型的句法产生式。它们可以覆盖指代：

* 序列类型（[tuple], [array], [slice]）。
* [类型路径(type paths)][Type paths]，这些包括：
    * 原生类型（[布尔型][boolean], [数字类类型][numeric], [文本类类型][textual]）。
    * 指向[程序项][item]（[结构体(`struct`)][struct], [枚举(`enum`)][enum], [联合体(`union`)][union], [类型别名][type alias], [trait]）的路径。
    * [`Self`路径][`Self` path]，其中 `Self` 是实现类型。
    * 泛型[类型参数][type parameters]。
* 指针类型（[引用][reference], [裸指针][raw pointer], [函数指针][function pointer]）。
* [自动推断型类型(inferred type)][inferred type]，就是请求编译器确定类型的类型。
* 用来消除歧义的[圆括号][Parentheses]。
* Trait类型：[trait对象(trait object)][Trait objects] 和 [实现trait(impl trait)][impl trait].
* [never]型(`!`)。
* 展开成类型表达式的[宏][Macros]。

### 圆括号组合类型

> _ParenthesizedType_ :\
> &nbsp;&nbsp; `(` [_Type_] `)`

在某些情况下，类型组合在一起时可能会产生二义性。此时可以在类型周围使用元括号来避免歧义。例如，[引用类型][reference type]的[类型约束][type boundaries]列表中的 `+`运算符搞不清楚其左值的边界位置在哪里，因此需要使用圆括号来明确其边界。这里需要的消歧文法就是使用 [_TypeNoBounds_] 句法规则替代 [_Type_] 句法规则。

```rust
# use std::any::Any;
type T<'a> = &'a (dyn Any + Send);
```

## 递归类型

标称类型(nominal types) &mdash; [结构体(`struct`)][structs]、[枚举(`enum`)][enumerations]和[联合体(`union`)][unions] &mdash; 可以是递归的。也就是说，每个枚举(`enum`)变体或结构体(`struct`)或联合体(`union`)的字段可以直接或间接地指向此枚举(`enum`)或结构体(`struct`)类型本身。这种递归有一些限制：

* 递归类型必须在递归中包含一个标称类型（不能仅是[类型别名][type aliases]或其他结构化的类型，如[数组][arrays]或[元组][tuples]）。因此不允许使用 `type Rec = &'static [Rec]`。
* 递归类型的尺寸必须是有限的；也就是说，类型的递归字段必须是[指针类型][pointer types]。
* 递归类型的定义可以跨越模块边界，但不能跨越模块*可见性*边界或 crate 边界（为了简化模块系统和类型检查）。

*递归*类型及使用示例：

```rust
enum List<T> {
    Nil,
    Cons(T, Box<List<T>>)
}

let a: List<i32> = List::Cons(7, Box::new(List::Cons(13, Box::new(List::Nil))));
```

[_ArrayType_]: types/array.md
[_BareFunctionType_]: types/function-pointer.md
[_ImplTraitTypeOneBound_]: types/impl-trait.md
[_ImplTraitType_]: types/impl-trait.md
[_InferredType_]: types/inferred.md
[_MacroInvocation_]: macros.md#macro-invocation
[_NeverType_]: types/never.md
[_ParenthesizedType_]: types.md#parenthesized-types
[_QualifiedPathInType_]: paths.md#qualified-paths
[_RawPointerType_]: types/pointer.md#raw-pointers-const-and-mut
[_ReferenceType_]: types/pointer.md#shared-references-
[_SliceType_]: types/slice.md
[_TraitObjectTypeOneBound_]: types/trait-object.md
[_TraitObjectType_]: types/trait-object.md
[_TupleType_]: types/tuple.md#tuple-types
[_TypeNoBounds_]: types.md#type-expressions
[_TypePath_]: paths.md#paths-in-types
[_Type_]: types.md#type-expressions

[Array]: types/array.md
[Boolean]: types/boolean.md
[Closures]: types/closure.md
[Enum]: types/enum.md
[Function pointers]: types/function-pointer.md
[Functions]: types/function-item.md
[Impl trait]: types/impl-trait.md
[Macros]: macros.md
[Numeric]: types/numeric.md
[Parentheses]: #parenthesized-types
[Raw pointers]: types/pointer.md#raw-pointers-const-and-mut
[References]: types/pointer.md#shared-references-
[Slice]: types/slice.md
[Struct]: types/struct.md
[Textual]: types/textual.md
[Trait objects]: types/trait-object.md
[Tuple]: types/tuple.md
[Type paths]: paths.md#paths-in-types
[Union]: types/union.md
[`Self` path]: paths.md#self-1
[arrays]: types/array.md
[enumerations]: types/enum.md
[function pointer]: types/function-pointer.md
[inferred type]: types/inferred.md
[item]: items.md
[never]: types/never.md
[pointer types]: types/pointer.md
[raw pointer]: types/pointer.md#raw-pointers-const-and-mut
[reference type]: types/pointer.md#shared-references-
[reference]: types/pointer.md#shared-references-
[structs]: types/struct.md
[trait]: types/trait-object.md
[tuples]: types/tuple.md
[type alias]: items/type-aliases.md
[type aliases]: items/type-aliases.md
[type boundaries]: trait-bounds.md
[type parameters]: types/parameters.md
[unions]: types/union.md

<!-- 2020-11-12-->
<!-- checked -->
