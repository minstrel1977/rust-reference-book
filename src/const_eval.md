# Constant evaluation
# 常量求值

>[const_eval.md.md](https://github.com/rust-lang/reference/blob/master/src/const_eval.md.md)\
>commit  7ad799da00dd162638999e38dad10905bf6c7ec6

常量求值是在编译过程中计算[表达式][[expressions]]结果的过程。只有全部表达式形式的一个子集可以在编译时求值。

## Constant expressions
## 常量表达式

某些形式的表达式(称为常量表达式)可以在编译时求值。在[常量(const)上下文](#const-context)中，常量表达式是唯一允许的表达式，并且总是在编译时求值。在其他地方(比如 [let语句][let statements])，常量表达式*可以*在编译时求值，但不能保证总是这样。如果值必须在编译时求得(例如在常量上下文中)，则像[数组索引][array indexing]越界或[溢出][overflow]这样的行为都是编译错误。如果不是必须在编译时求值，则这些行为在编译时只是警告（但在运行时可能会引起 panic）。

只要它们的所有操作数也都是常量表达式，并且在求值后不会运行任何 [`Drop::drop`][destructors] 函数，那下列表达式就是常量表达式。

* [字面量][Literals]。
* [函数][functions]和[常量项][constants]的[路径][Paths]。
  Recursively defining constants is not allowed.
* Paths to [statics]. These are only allowed within the initializer of a static.
* [Tuple expressions].
* [Array expressions].
* [Struct] expressions.
* [Enum variant] expressions.
* [Block expressions], including `unsafe` blocks.
    * [let statements] and thus irrefutable [patterns], including mutable bindings
    * [assignment expressions]
    * [compound assignment expressions]
    * [expression statements]
* [Field] expressions.
* Index expressions, [array indexing] or [slice] with a `usize`.
* [Range expressions].
* [Closure expressions] which don't capture variables from the environment.
* Built-in [negation], [arithmetic], [logical], [comparison] or [lazy boolean]
  operators used on integer and floating point types, `bool`, and `char`.
* Shared [borrow]s, except if applied to a type with [interior mutability].
* The [dereference operator] except for raw pointers.
* [Grouped] expressions.
* [Cast] expressions, except
  * pointer to address casts,
  * function pointer to address casts, and
  * unsizing casts to trait objects.
* Calls of [const functions] and const methods.
* [loop], [while] and [`while let`] expressions.
* [if], [`if let`] and [match] expressions.

## Const context

A _const context_ is one of the following:

* [Array type length expressions]
* [Array repeat length expressions][array expressions]
* The initializer of
  * [constants]
  * [statics]
  * [enum discriminants]

## Const Functions

A _const fn_ is a function that one is permitted to call from a const context. Declaring a function
`const` has no effect on any existing uses, it only restricts the types that arguments and the
return type may use, as well as prevent various expressions from being used within it. You can freely do anything with a const function that
you can do with a regular function.

When called from a const context, the function is interpreted by the
compiler at compile time. The interpretation happens in the
environment of the compilation target and not the host. So `usize` is
`32` bits if you are compiling against a `32` bit system, irrelevant
of whether you are building on a `64` bit or a `32` bit system.

Const functions have various restrictions to make sure that they can be
evaluated at compile-time. It is, for example, not possible to write a random
number generator as a const function. Calling a const function at compile-time
will always yield the same result as calling it at runtime, even when called
multiple times. There's one exception to this rule: if you are doing complex
floating point operations in extreme situations, then you might get (very
slightly) different results. It is advisable to not make array lengths and enum
discriminants depend on floating point computations.


Notable features that const contexts have, but const fn haven't are:

* floating point operations
  * floating point values are treated just like generic parameters without trait bounds beyond
  `Copy`. So you cannot do anything with them but copy/move them around.
* `dyn Trait` types
* generic bounds on generic parameters beyond `Sized`
* comparing raw pointers
* union field access
* [`transmute`] invocations.

Conversely, the following are possible in a const function, but not in a const context:

* Use of generic parameters.

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


使用关键字 `const` 限定的函数是常量函数，与[元组结构体]和[元祖变体]构造函数一样。*常量函数*可以在[常量上下文]中调用。当从常量上下文中调用这类函数时，编译器会在编译时解释该函数。这种解释发生在(Rust编译器为)编译目标(构建的模拟)环境中，而不是在当前主机环境中。因此，如果您是针对一个 `32` 位系统进行编译，那么 `usize` 就是 `32` 位，这与您在一个 `64` 位还是在一个 `32` 位系统上进行编译无关。

如果在[常量上下文]之外调用常量函数，那么它与任何其他函数没有区别。你可以自由地用常量函数做任何你可以用普通函数做的任何事情。

常量函数有各种限制以确保其可以在编译时对其求值。例如，不可以将随机数生成器编写为常量函数。在编译时调用常量函数将始终产生与运行时调用它相同的结果，即使多次调用也是如此。这个规则有一个例外：如果您在极端情况下执行复杂的浮点运算，那么您可能得到（非常轻微）不同的结果。建议不要使数组长度和枚举判别式依赖于浮点计算。

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
* `unsafe` 块和 `const unsafe fn` 可以，但代码体/块只能执行以下非安全操作：
    * 调用非安全常量函数
