# Constant evaluation
# 常量求值

>[const_eval.md](https://github.com/rust-lang/reference/blob/master/src/const_eval.md)\
>commit:  ace105a62d42374682b2dc2b6f63a1aa6814ba91 \
>本章译文最后维护日期：2024-10-13

常量求值是在编译过程中计算[表达式][expressions]结果的过程。（不是所有表达式都可以在编译时求值，也就是说）只有全部表达式的某个子集可以在编译时求值。

## Constant expressions
## 常量表达式

某些形式的表达式（被称为常量表达式）可以在编译时求值。在[常量(const)上下文](#const-context)中，常量表达式是唯一允许的表达式，并且总是在编译时求值。在其他地方，比如 [let语句][let statements]，常量表达式*可以*在编译时求值，但不能保证总能在此时求值。如果值必须在编译时求得（例如在常量上下文中），则像[数组索引][array indexing]越界或[溢出][overflow]这样的行为都是编译错误。如果不是必须在编译时求值，则这些行为在编译时只是警告，但它们在运行时可能会触发 panic。

下列表达式中，只要它们的所有操作数都是常量表达式，并且求值/计算不会引起任何 [`Drop::drop`][destructors]函数的运行，那这些表达式就是常量表达式。

* [字面量][Literals]。
* [常量参数][Const parameters]。
* 指向[函数项][functions]和[常量项][constants]的[路径][Paths]。不允许递归地定义常量项。
* 指向[静态项][statics]的路径。这种路径只允许出现在静态项的初始化器中。
* [元组表达式][Tuple expressions]。
* [数组表达式][Array expressions]。
* [结构体][Struct]表达式。
* [块表达式][Block expressions]，包括`unsafe`块和`const`块。
    * [let语句][let statements]以及类似这样的不可反驳型[模式][patterns]绑定，包括可变绑定。
    * [赋值表达式][assignment expressions]
    * [复合赋值表达式][compound assignment expressions]
    * [表达式语句][expression statements]
* [字段][Field]表达式。
* 索引表达式，长度为 `usize` 的[数组索引][array indexing]或[切片][slice]。
* [区间表达式][Range expressions]。
* 未从环境捕获变量的[闭包][Closure expressions]。
* 在整型、浮点型、布尔型(`bool`)和字符型(`char`)上做的各种内置运算，包括：[取反][negation]、[算术][arithmetic]、[逻辑][logical]、[比较][comparison] 或 [惰性布尔][lazy boolean]运算。
* 所有形式的[借用][borrow]，包括裸指针借用，都有一个限制：
  可变借用和共享借用到具有内部可变性的值时，仅允许引用“瞬态(transient)”位置。如果一个位置的生存期严格包含在当前[常量上下文][const context]中，则该位置是*瞬态(transient)的*。
* [解引用操作][dereference operator]
  * 指针到地址的强制转换，
  * 函数指针到地址的强制转换，和
  * 到 trait对象的非固定内存宽度类型强换(unsizing casts)。
* 调用[常量函数][const functions]和常量方法。
* [loop], [while] 和 [`while let`] 表达式。
* [if], [`if let`] 和 [匹配(match)] 表达式。

## Const context
## 常量上下文

下述位置是*常量上下文*：

* [数组类型的长度表达式][Array type length expressions]
* [分号分隔的数组创建形式中的长度表达式][array expressions]
* 下述表达式的初始化器(initializer)：
  * [常量项][constants]
  * [静态项][statics]
  * [枚举判别值][enum discriminants]
* [常量型泛型实参][const generic argument]
* [常量块][const block]

用作类型的一部分的常量上下文（数组类型和重复长度表达式以及常量泛型参数）只能限制使用周围的泛型参数：这样的表达式必须是单个裸露的常量泛型参数，或者是不使用任何泛型的任意表达式。

## Const Functions
## 常量函数

*常量函数(const fn)*可以在常量上下文中调用。给一个函数加一个常量(`const`)标志对该函数的任何现有的使用都没有影响，它只限制参数和返回可以使用的类型，并将函数体限制为常量表达式。

当从常量上下文中调用这类函数时，编译器会在编译时解释该函数。这种解释发生在编译目标环境中，而不是在当前主机环境中。因此，如果是针对一个 `32` 位目标系统进行编译，那么 `usize` 就是 `32` 位，这与在一个 `64` 位还是在一个 `32` 位主机环境中进行编译动作无关。

[arithmetic]:           expressions/operator-expr.md#arithmetic-and-logical-binary-operators
[array expressions]:    expressions/array-expr.md
[array indexing]:       expressions/array-expr.md#array-and-slice-indexing-expressions
[array indexing]:       expressions/array-expr.md#array-and-slice-indexing-expressions
[array type length expressions]: types/array.md
[assignment expressions]: expressions/operator-expr.md#assignment-expressions
[compound assignment expressions]: expressions/operator-expr.md#compound-assignment-expressions
[block expressions]:    expressions/block-expr.md
[borrow]:               expressions/operator-expr.md#borrow-operators
[cast]:                 expressions/operator-expr.md#type-cast-expressions
[closure expressions]:  expressions/closure-expr.md
[comparison]:           expressions/operator-expr.md#comparison-operators
[const block]:          expressions/block-expr.md#const-blocks
[const functions]:      items/functions.md#const-functions
[const generic argument]: items/generics.md#const-generics
[const generic parameters]: items/generics.md#const-generics
[constants]:            items/constant-items.md
[Const parameters]:     items/generics.md
[dereference operator]: expressions/operator-expr.md#the-dereference-operator
[destructors]:          destructors.md
[enum discriminants]:   items/enumerations.md#discriminants
[expression statements]: statements.md#expression-statements
[expressions]:          expressions.md
[field]:                expressions/field-expr.md
[functions]:            items/functions.md
[grouped]:              expressions/grouped-expr.md
[interior mutability]:  interior-mutability.md
[if]:                   expressions/if-expr.md#if-expressions
[`if let`]:             expressions/if-expr.md#if-let-expressions
[lazy boolean]:         expressions/operator-expr.md#lazy-boolean-operators
[let statements]:       statements.md#let-statements
[literals]:             expressions/literal-expr.md
[logical]:              expressions/operator-expr.md#arithmetic-and-logical-binary-operators
[loop]:                 expressions/loop-expr.md#infinite-loops
[match]:                expressions/match-expr.md
[negation]:             expressions/operator-expr.md#negation-operators
[overflow]:             expressions/operator-expr.md#overflow
[paths]:                expressions/path-expr.md
[patterns]:             patterns.md
[range expressions]:    expressions/range-expr.md
[slice]:                types/slice.md
[statics]:              items/static-items.md
[struct]:               expressions/struct-expr.md
[tuple expressions]:    expressions/tuple-expr.md
[while]:                expressions/loop-expr.md#predicate-loops
[`while let`]:          expressions/loop-expr.md#predicate-pattern-loops
