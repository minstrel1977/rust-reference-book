# Constant evaluation
# 常量求值

>[const_eval.md](https://github.com/rust-lang/reference/blob/master/src/const_eval.md)\
>commit:  7ad799da00dd162638999e38dad10905bf6c7ec6

常量求值是在编译过程中计算[表达式][[expressions]]结果的过程。只有全部表达式形式的一个子集可以在编译时求值。

## Constant expressions
## 常量表达式

某些形式的表达式(称为常量表达式)可以在编译时求值。在[常量(const)上下文](#const-context)中，常量表达式是唯一允许的表达式，并且总是在编译时求值。在其他地方(比如 [let语句][let statements])，常量表达式*可以*在编译时求值，但不能保证总是这样。如果值必须在编译时求得(例如在常量上下文中)，则像[数组索引][array indexing]越界或[溢出][overflow]这样的行为都是编译错误。如果不是必须在编译时求值，则这些行为在编译时只是警告（但在运行时可能会引起 panic）。

只要它们的所有操作数也都是常量表达式，并且在求值后不会运行任何 [`Drop::drop`][destructors] 函数，那下列表达式就是常量表达式。

* [字面量][Literals]。
* [函数][functions]和[常量项][constants]的[路径][Paths]。不允许递归定义常量项。
* [静态项][statics]的路径。这种路径只允许出现在静态项的初始值设定中。
* [元组表达式][Tuple expressions]。
* [数组表达式][Array expressions]。
* [结构体(`struct`)][Struct]表达式。
* [枚举变体][Enum variant]表达式。
* [块表达式][Block expressions]，包括非安全(`unsafe`)块。
    * [let语句][let statements]以及类似这样的不可反驳的[模式][patterns]绑定，包括可变绑定。
    * [赋值表达式][assignment expressions]
    * [复合赋值表达式][compound assignment expressions]
    * [表达式语句][expression statements]
* [字段][Field]表达式。
* 索引表达式，[数组索引][array indexing]或 `usize` 长度的[切片][slice]。
* [区间表达式][Range expressions]。
* 未从环境捕获变量的[闭包][Closure expressions]。
* 使用在整型、浮点型、布尔型(`bool`)和字符型(`char`)上的内置的[取反][negation]，[算术][arithmetic]，[逻辑][logical]，[比较][comparison] 或 [惰性布尔][lazy boolean]运算符表达式。
* 共享[借用][borrow]，排除借用类型为[内部可变性][interior mutability]的情况。
* [解引用操作符][dereference operator]，排除解引用裸指针的情况。
* [分组][Grouped]表达式。
* [强制转换][Cast]表达式，排除
  * 指针到地址的强制转换，
  * 函数指针到地址的强制转换，和
  * 到 trait对象的非固定尺寸类型强制转换。
* 调用[常量函数][const functions]和常量方法。
* [loop], [while] 和 [`while let`] 表达式。
* [if], [`if let`] 和 [match] 表达式。

## Const context
## 常量上下文

*常量上下文*是下述表达式之一：

* [数组类型内的数组长度表达式][Array type length expressions]
* [逗号分隔的数组表达式][array expressions]
* 下述表达式的初始化器(initializer)：
  * [常量项][constants]
  * [静态项][statics]
  * [枚举判别值][enum discriminants]

## Const Functions
## 常量函数

*常量函数*可以在常量上下文中调用。给一个函数加一个常量标志(`const`)对该函数的任何现有的使用都没有影响，它只限制参数和返回类型可以使用的类型，并防止在这两个位置上使用不被允许的表达式类型。你可以自由地用常量函数去做任何你可以用常规函数做的事情。

当从常量上下文中调用这类函数时，编译器会在编译时解释该函数。这种解释发生在(Rust编译器为)构建目标(构建的模拟编译)环境中，而不是在当前主机环境中。因此，如果您是针对一个 `32` 位系统进行编译，那么 `usize` 就是 `32` 位，这与您在一个 `64` 位还是在一个 `32` 位系统上进行编译无关。

常量函数有各种限制以确保其可以在编译时对其求值。因此，例如，不可以将随机数生成器编写为常量函数。在编译时调用常量函数将始终产生与运行时调用它相同的结果，即使多次调用也是如此。这个规则有一个例外：如果您在极端情况下执行复杂的浮点运算，那么您可能得到（非常轻微）不同的结果。建议不要使数组长度和枚举判别式依赖于浮点计算。

常量上下文有，但常量函数不具备的显著特性有：

* 浮点运算
  * 处理浮点值就像处理只有 `Copy` 这个trait约束的泛型参数一样，你不能用它们做任何事，只能复制/移动它们。
* `dyn Trait` 类型
* `Sized` 泛型约束之外的泛型约束
* 比较裸指针
* 访问联合体字段
* 调用 [`transmute`]。

相反地，以下情况在常量函数中是可能的，但在常量上下文中则不可能：

* 使用泛型参数。

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
[const functions]:      items/functions.md#const-functions
[constants]:            items/constant-items.md
[dereference operator]: expressions/operator-expr.md#the-dereference-operator
[destructors]:          destructors.md
[enum discriminants]:   items/enumerations.md#custom-discriminant-values-for-fieldless-enumerations
[enum variant]:         expressions/enum-variant-expr.md
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
[`transmute`]:          ../std/mem/fn.transmute.html
[while]:                expressions/loop-expr.md#predicate-loops
[`while let`]:          expressions/loop-expr.md#predicate-pattern-loops



允许出现在常量函数中的数据结构的详尽列表：
<!--Exhaustive list of permitted structures in const functions: TobeModify-->

> **注意**: 这个列表比你可以用常规常量写的东西更具限制性

* 类型参数中参数只能有以下类型的 [trait约束]：
<!--* Type parameters where the parameters only have any [trait bounds] of the following kind: TobeModify-->
    * 生存期
    * `Sized` 或 [`?Sized`]

    这意味着 `<T: 'a + ?Sized>`、`<T: 'b + Sized>` 和 `<T>` 都是可以的。
    
    此规则也适用于包含常量方法的 *impl 块*的类型参数。
    
    此规则不适用于元组结构体和元组变体构造函数。    

* 整型上的算术和比较运算符
* 所有布尔运算符，包括 `&&` 和 `||`
* 任何类型的聚合构造函数（数组、`struct`、`enum`、元组，…）
* 对其他*安全*常量函数的调用（无论是通过函数调用还是通过方法调用）
* 数组和切片上的索引表达式
* 对结构体和元组的字段的访问
* 从常量项（但不能是静态项，甚至不能引用静态项）中读取数据
* `&` 和 `*`（仅解引用引用，原始指针不行）
* `if`、`if let`、和 `match`
* `while`、`while let`、和 `loop`
* 除裸指针向整型和切片转换以外的类型转换
*非安全(`unsafe`)块和 `const unsafe fn` 可以，但代码体/块只能执行以下非安全操作：
    * 调用非安全常量函数
