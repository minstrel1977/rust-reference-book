# Items
# 数据项

>[items.md](https://github.com/rust-lang/reference/blob/master/src/items.md)\
>commit: b0e0ad6490d6517c19546b1023948986578fc378 \
>本译文最后维护日期：2020-10-19

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

*数据项*[^译者注]是 crate 的组成部分。数据项由一套嵌套的[模块][modules]被组织在一个 crate 内。每个 crate 都有一个“最外层”的匿名模块；crate 中的所有数据项都在 crate 的模块树中有它们的[路径][paths]。

数据项在编译时就完全确定下来了，通常在执行期间保持固定，并可能驻留在只读内存中。
An _item_ is a component of a crate. Items are organized within a crate by a nested set of [modules]. Every crate has a single "outermost" anonymous module; all further items within the crate have [paths] within the module tree of the crate.

Items are entirely determined at compile-time, generally remain fixed during
execution, and may reside in read-only memory.

There are several kinds of items:

* [modules]
* [`extern crate` declarations]
* [`use` declarations]
* [function definitions]
* [type definitions]
* [struct definitions]
* [enumeration definitions]
* [union definitions]
* [constant items]
* [static items]
* [trait definitions]
* [implementations]
* [`extern` blocks]

Some items form an implicit scope for the declaration of sub-items. In other
words, within a function or module, declarations of items can (in many cases)
be mixed with the statements, control blocks, and similar artifacts that
otherwise compose the item body. The meaning of these scoped items is the same
as if the item was declared outside the scope &mdash; it is still a static item
&mdash; except that the item's *path name* within the module namespace is
qualified by the name of the enclosing item, or is private to the enclosing
item (in the case of functions). The grammar specifies the exact locations in
which sub-item declarations may appear.
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

<!-- 2020-10-16 -->
<!-- checked -->
