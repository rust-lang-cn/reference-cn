# 项

> **<sup>Syntax:<sup>**\
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
> &nbsp;&nbsp; )`
>
> _MacroItem_:\
> &nbsp;&nbsp; &nbsp;&nbsp; [_MacroInvocationSemi_]\
> &nbsp;&nbsp; | [_MacroRulesDefinition_]

_项（item）_ 是 crate 的一个组件。项在 crate 中被组织为一个嵌套的[模块][modules]集。每个 crate 都有一个单独的“最外层”的匿名模块；crate 中的所有其它项，在 crate 中的模块树中都有路径。

项全部在编译时被确定，大部分在执行期间保持不变，并可能驻留在只读内存中。

项有以下几种类型：

* [模块][modules]
* [extern crate 声明][`extern crate` declarations]
* [use 声明][`use` declarations]
* [函数定义][function definitions]
* [类型定义][type definitions]
* [结构体定义][struct definitions]
* [枚举定义][enumeration definitions]
* [联合体定义][union definitions]
* [常量项][constant items]
* [静态项][static items]
* [trait 定义][trait definitions]
* [实现][implementations]
* [`外部`块][`extern` blocks]

这些范围内的项与定义在作用域外声明的项是一样的--它仍是一个静态项--除了那些位于模块命名空间中的路径名称由包含其中的项的名字定义的项，或者是其包含的私有项（对于函数来说）。

有些项组成了子项声明的隐式作用域。换句话说，在一个函数或模块中，项的声明可以（在许多情况下）与语句、控制块和类似的构件混合在一起，否则这些构件将构成项体。这些作用域内的项的含义与在作用域外声明的项的含义相同——它仍然是静态项——只是模块命名空间中的项的 *路径名* 由包含该项的封闭项名称限定，或者是包含该项的封闭项的私有项（对于函数来说）。由语法指定子项声明可能出现的确切位置。

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
