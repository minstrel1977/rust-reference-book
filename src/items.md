# 数据项

>[items.md](https://github.com/rust-lang/reference/blob/master/src/items.md)\
>commit: b0e0ad6490d6517c19546b1023948986578fc378

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

*数据项*[^译者注]是 crate 的组成部分。数据项通过一套嵌套的[模块]被组织在一个 crate 内由。每个 crate 都有一个“最外层”的匿名模块；crate 中的所有数据项都在 crate 的模块树中有它们的[路径]。rate.

数据项在编译时就完全确定下来了，通常在执行期间保持固定，并可能驻留在只读内存中。

有几类数据项:

* [模块]
* [`extern crate` 声明]
* [`use` 声明]
* [函数定义]
* [类型定义]
* [结构体定义]
* [枚举定义]
* [联合体定义]
* [常量项]
* [静态项]
* [trait 定义]
* [实现]
* [`extern` 块]

有些数据项会形成子项声明的隐式作用域。换句话说，在一个函数或模块中，数据项的声明可以（在许多情况下）与语句、控制块以及类似的能构成数据项的构件。这些在作用域内的数据项的含义与在作用域外声明的数据项的含义相同（它仍然是静态项），只是该数据项在模块的命名空间中的*路径名*由封闭它数据项的名称限定，当然该数据项也可能是封闭它的数据项的私有数据项（在函数的情况下）。语法规范会指定子项声明可能出现的合法位置。

[^译者注]:为方便汉语阅读和避免汉语中的歧义，单独出现item时，译者一般翻译为“数据项”，但和其他名词复合时，又一般会译为“XX项”。

[_ConstantItem_]: items/constant-items.md
[_Enumeration_]: items/enumerations.md
[_ExternBlock_]: items/external-blocks.md
[_ExternCrate_]: items/extern-crates.md
[_Function_]: items/functions.md
[_Implementation_]: items/implementations.md
[_MacroInvocationSemi_]: macros.md#宏调用
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
[`extern crate` 声明]: items/extern-crates.md
[`extern` 块]: items/external-blocks.md
[`use` 声明]: items/use-declarations.md
[常量项]: items/constant-items.md
[枚举定义]: items/enumerations.md
[函数定义]: items/functions.md
[实现]: items/implementations.md
[模块]: items/modules.md
[路径]: paths.md
[静态项]: items/static-items.md
[结构体定义]: items/structs.md
[trait 定义]: items/traits.md
[类型定义]: items/type-aliases.md
[联合体定义]: items/unions.md
