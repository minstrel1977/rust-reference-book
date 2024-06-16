# Items
# 程序项

>[items.md](https://github.com/rust-lang/reference/blob/master/src/items.md)\
>commit: 524b9b1efdfdef603d9617d8e1476b66b99a6349 \
>本章译文最后维护日期：2024-06-15

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

程序项可以声明在[crate的根层][root of the crate]中、[模块][modules]中或一个[块表达式][block expression]中。
[关联程序项][associated items]，程序项的的一种子集，可以声明在 [traits] 和 [实现][implementations]中。
外部程序项，程序项的的一种子集，它可以声明在[外部块][`extern` blocks]中。

程序项可以以任意顺序定义，但 [`macro_rules`] 例外，它有自己特有的作用域行为表象。
程序项名称的[名称解析][Name resolution]规则允许在模块或块中引用程序项的位置之前或之后定义该程序项。

有关程序项的作用域规则的信息，请参阅[程序项作用域][item scopes]章节。

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
[`macro_rules`]: macros-by-example.md
[`use` declarations]: items/use-declarations.md
[associated items]: items/associated-items.md
[block expression]: expressions/block-expr.md
[constant items]: items/constant-items.md
[enumeration definitions]: items/enumerations.md
[function definitions]: items/functions.md
[implementations]: items/implementations.md
[item scopes]: names/scopes.md#item-scopes
[modules]: items/modules.md
[name resolution]: names/name-resolution.md
[paths]: paths.md
[root of the crate]: crates-and-source-files.md
[statement]: statements.md
[static items]: items/static-items.md
[struct definitions]: items/structs.md
[trait definitions]: items/traits.md
[traits]: items/traits.md
[type definitions]: items/type-aliases.md
[union definitions]: items/unions.md
