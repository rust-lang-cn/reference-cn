# 程序项

>[items.md](https://github.com/rust-lang/reference/blob/master/src/items.md)\
>commit: b0e0ad6490d6517c19546b1023948986578fc378 \
>本章译文最后维护日期：2020-11-8

> **<sup>句法:<sup>**\
> _Item_:\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp; _VisItem_\
> &nbsp;&nbsp; | _MacroItem_
>
> _VisItem_:\
> &nbsp;&nbsp; [_Visibility_]<sup>?</sup>\
> &nbsp;&nbsp; (\
> &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;  [_Module_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_ExternCrate_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_UseDeclaration_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_Function_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_TypeAlias_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_Struct_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_Enumeration_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_Union_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_ConstantItem_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_StaticItem_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_Trait_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_Implementation_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_ExternBlock_]\
> &nbsp;&nbsp; )
>
> _MacroItem_:\
> &nbsp;&nbsp; &nbsp;&nbsp; [_MacroInvocationSemi_]\
> &nbsp;&nbsp; | [_MacroRulesDefinition_]

*[程序项](翻译说明.md#常用词翻译)*是 crate 的组成单元。程序项由一套嵌套的[模块][modules]被组织在一个 crate 内。每个 crate 都有一个“最外层”的匿名模块；crate 中所有的程序项都在其 crate 的模块树中自己的[路径][paths]。

程序项在编译时就完全确定下来了，通常在执行期间保持结构稳定，并可以驻留在只读内存中。

有以下几类程序项:

* [模块][modules]
* [外部crate(`extern crate`)声明][`extern crate` declarations]
* [`use`声明][`use` declarations]
* [函数定义][function definitions]
* [类型定义][type definitions]
* [结构体定义][struct definitions]
* [枚举定义][enumeration definitions]
* [联合体定义][union definitions]
* [常量项][constant items]
* [静态项][static items]
* [trait定义][trait definitions]
* [实现][implementations]
* [外部块(`extern` blocks)][`extern` blocks]

有些程序项会形成子（数据）项声明的隐式作用域。换句话说，在一个函数或模块中，程序项的声明可以（在许多情况下）与语句、控制块、以及类似的能构成程序项主体的部件混合在一起。这些在作用域内的程序项的意义与在作用域外声明的程序项的意义相同（它仍然是静态项），只是该程序项在模块的命名空间中的*路径名*由封闭它的程序项的名称限定，或该程序项也可能是封闭它的程序项的私有程序项（比如函数的情况）。语法规范指定了子项声明可能出现的合法位置。

[_ConstantItem_]: items/constant-items.md
[_Enumeration_]: items/enumerations.md
[_ExternBlock_]: items/external-blocks.md
[_ExternCrate_]: items/extern-crates.md
[_Function_]: items/functions.md
[_Implementation_]: items/implementations.md
[_MacroInvocationSemi_]: macros.md#macro-invocation
[_MacroRulesDefinition_]: macros-by-example.md
[_Module_]: items/modules.md
[_OuterAttribute_]: attributes.md
[_StaticItem_]: items/static-items.md
[_Struct_]: items/structs.md
[_Trait_]: items/traits.md
[_TypeAlias_]: items/type-aliases.md
[_Union_]: items/unions.md
[_UseDeclaration_]: items/use-declarations.md
[_Visibility_]: visibility-and-privacy.md
[`extern crate` declarations]: items/extern-crates.md
[`extern` blocks]: items/external-blocks.md
[`use` declarations]: items/use-declarations.md
[constant items]: items/constant-items.md
[enumeration definitions]: items/enumerations.md
[function definitions]: items/functions.md
[implementations]: items/implementations.md
[modules]: items/modules.md
[paths]: paths.md
[static items]: items/static-items.md
[struct definitions]: items/structs.md
[trait definitions]: items/traits.md
[type definitions]: items/type-aliases.md
[union definitions]: items/unions.md

<!-- 2020-11-12-->
<!-- checked -->
