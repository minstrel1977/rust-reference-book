# 数据项

>[items.md](https://github.com/rust-lang/reference/blob/master/src/items.md)\
>commit b0e0ad6490d6517c19546b1023948986578fc378

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

*数据项*是 crate 的组成部分。数据项通过一套嵌套的[模块]被组织在一个 crate 内由。每个 crate 都有一个“最外层”的匿名模块；crate 中的所有数据项都在 crate 的模块树中有它们的[路径]。rate.

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
* [常量数据项]
* [静态数据项]
* [trait 定义]
* [实现]
* [`extern` 块]

Some items form an implicit scope for the declaration of sub-items. In other words, within a function or module, declarations of items can (in many cases) be mixed with the statements, control blocks, and similar artifacts that otherwise compose the item body. The meaning of these scoped items is the same as if the item was declared outside the scope - it is still a static item - except that the item's *path name* within the module namespace is qualified by the name of the enclosing item, or is private to the enclosing item (in the case of functions). The grammar specifies the exact locations in which sub-item declarations may appear.（译者注：这一段译者没看懂，回头懂了再来翻译，先在这里打个标记TobeModify）

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
[`extern crate` 声明]: items/extern-crates.md
[`extern` 块]: items/external-blocks.md
[`use` 声明]: items/use-declarations.md
[常量数据项]: items/constant-items.md
[枚举定义]: items/enumerations.md
[函数定义]: items/functions.md
[实现]: items/implementations.md
[模块]: items/modules.md
[路径]: paths.md
[静态数据项]: items/static-items.md
[结构体定义]: items/structs.md
[trait 定义]: items/traits.md
[类型定义]: items/type-aliases.md
[联合体定义]: items/unions.md
