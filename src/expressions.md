# 表达式

>[expressions.md](https://github.com/rust-lang/reference/blob/master/src/expressions.md)\
>commit 8c4522851452563b715b11d4cd755b36d8e4bca5

> **<sup>句法</sup>**\
> _Expression_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _ExpressionWithoutBlock_\
> &nbsp;&nbsp; | _ExpressionWithBlock_
>
> _ExpressionWithoutBlock_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup>[†](#表达式属性)\
> &nbsp;&nbsp; (\
> &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; [_LiteralExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_PathExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_OperatorExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_GroupedExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_ArrayExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_AwaitExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_IndexExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_TupleExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_TupleIndexingExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_StructExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_EnumerationVariantExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_CallExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_MethodCallExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_FieldExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_ClosureExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_ContinueExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_BreakExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_RangeExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_ReturnExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_MacroInvocation_]\
> &nbsp;&nbsp; )
>
> _ExpressionWithBlock_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup>[†](#表达式属性)\
> &nbsp;&nbsp; (\
> &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; [_BlockExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_AsyncBlockExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_UnsafeBlockExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_LoopExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_IfExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_IfLetExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_MatchExpression_]\
> &nbsp;&nbsp; )

一个表达式可能有两个角色：它总是产生一个*值*；它可能有*效果*（或者称为“副作用”）。表达式的*计算结果为*值，并在*求值*（译者注：有时也称对表达式进行计算）期间起效果。许多表达式包含子表达式（操作数）。每种表达方式的含义都有以下几点：

* 在对表达式求值时是否对子表达式求值
* 对子表达式求值的顺序
* 如何组合子表达式的值来获得表达式的值

这样，表达式的结构决定了执行的结构。代码块只是另一种表达式，所以代码块、语句和表达式可以递归地彼此嵌套到任意深度。

## 表达式的优先级

Rust 运算符和表达式的优先级顺序如下，从强到弱。具有相同优先级的二元运算符按其结合(associativity)顺序做了分组。

| 运算符/表达式         | 结合性       |
|-----------------------------|---------------------|
| Paths（路径）                |                     |
| Method calls（方法调用）               |                     |
| Field expressions （字段表达式）          | 从左向右       |
| Function calls, array indexing（函数调用，数组索引） |                  |
| `?`                         |                     |
| Unary（一元运算符） `-` `*` `!` `&` `&mut` |                    |
| `as`                        | 从左向右       |
| `*` `/` `%`                 | 从左向右       |
| `+` `-`                     | 从左向右       |
| `<<` `>>`                   | 从左向右       |
| `&`                         | 从左向右       |
| `^`                         | 从左向右       |
| <code>&#124;</code>         | 从左向右       |
| `==` `!=` `<` `>` `<=` `>=` | 需要圆括号 |
| `&&`                        | 从左向右       |
| <code>&#124;&#124;</code>   | 从左向右       |
| `..` `..=`                  | 需要圆括号 |
| `=` `+=` `-=` `*=` `/=` `%=` <br> `&=` <code>&#124;=</code> `^=` `<<=` `>>=` | 从右向左 |
| `return` `break` closures（返回、中断、闭包）   |                     |

## 位置表达式和值表达式

表达式分为两大类：位置表达式和值表达式。同样的，在每个表达式中，子表达式可以出现在位置上下文或值上下文中。表达式的求值既取决于它自己的类别，也取决于它所处的上下文。

*位置表达式*是表示内存位置的表达式。所以位置表达式本质就是指向内存地址的[路径]，目前的它的形式包括：局部变量、[静态变量]、[解引用][deref] (`*expr`)、[索引数组]表达式(`expr[expr]`)、[字段]引用(`expr.f`) 和圆括号括起来的表达式。那除了上述形式外所有其他形式的表达式都是值表达式。

*值表达式*是表示实际值的表达式。

下面的上下文是*位置表达式*上下文：

* [赋值][assign]或[复合赋值]表达式的左操作数。
* 一元运算符[借用]或[解引用][deref]的操作数。
* 字段表达式的操作数。
* 数组索引表达式的索引操作数。
* 任何[隐式借用]的操作数。
* [let语句]的初始化表达式。
* [`if let`]、[`match`][match] 或 [`while let`] 表达式的[检验对象(scrutinee)]。
* 结构体表达式里的[函数式更新]的基(base)。

> 注意：历史上，位置表达式被称为 *lvalues*，值表达式被称为 *rvalues*。

### 移动和复制类型

当位置表达式在值表达式上下文中求值，或在模式中被值绑定时，这表示求出的值会*保存进*（held in）当前表达式代表的内存地址。如果该值的类型实现了 [`Copy`]，那么该值将被从原来的位置表达式（也可以理解为原来的内存位置）中复制过来。如果该值的类型没有实现 [`Copy`]，但实现了 [`Sized`]，那么就可以把该值从原来的位置表达式里移出（move out）（到新位置表达式中）。移出对位置表达式有如下限制：<!-- When a place expression is evaluated in a value expression context, or is bound by value in a pattern, it denotes the value held _in_ that memory location. If the type of that value implements [`Copy`], then the value will be copied. In the remaining situations if that type is [`Sized`], then it may be possible to move the value. Only the following place expressions may be moved out of: 这里直译后看不懂，意译又怕理解错误，只能先打个标记 TobeModif-->

* [变量]（译者注：位置表达式的一种）当前未被借用。
* [临时值](#临时位置)。
* 可以移出的位置表达式的字段且该字段没实现[`Drop`]。<!-- [Fields][field] of a place expression which can be moved out of and doesn't implement [`Drop`]. TobeModify-->
* 对可移出且类型为 [`Box<T>`] 的表达式做[解引用][deref]解出的结果。<!-- The result of [dereferencing][deref] an expression with type [`Box<T>`] and that can also be moved out of. TobeModify-->

移出对位置表达式求值的结果到局部变量中后，原来位置将被去初始化（deinitialized），并且在重新初始化之前无法再次读取。在所有其他情况下，尝试在值表达式上下文中使用位置表达式是错误的 感觉理解不对，需要修改 need to modify
移出计算为局部变量的位置表达式，
Moving out of a place expression that evaluates to a local variable, the location is deinitialized and cannot be read from again until it is reinitialized. In all other cases, trying to use a place expression in a value expression context is an error.

### Mutability

For a place expression to be [assigned][assign] to, mutably [borrowed][borrow],
[implicitly mutably borrowed], or bound to a pattern containing `ref mut` it
must be _mutable_. We call these *mutable place expressions*. In contrast,
other place expressions are called *immutable place expressions*.

The following expressions can be mutable place expression contexts:

* Mutable [variables], which are not currently borrowed.
* [Mutable `static` items].
* [Temporary values].
* [Fields][field], this evaluates the subexpression in a mutable place
  expression context.
* [Dereferences][deref] of a `*mut T` pointer.
* Dereference of a variable, or field of a variable, with type `&mut T`. Note:
  This is an exception to the requirement of the next rule.
* Dereferences of a type that implements `DerefMut`, this then requires that
  the value being dereferenced is evaluated is a mutable place expression context.
* [Array indexing] of a type that implements `IndexMut`, this
  then evaluates the value being indexed, but not the index, in mutable place
  expression context.

### 临时位置

在大多数位置表达式上下文中使用值表达式时，会创建一个临时的未命名内存位置，并将该值初始化到该内存位置，而表达式将求值结果在存放到该位置，除非把此表达式[提升]为 `static`。临时语句的[drop scope]通常是封闭语句的结尾。
When using a value expression in most place expression contexts, a temporary unnamed memory location is created initialized to that value and the expression evaluates to that location instead, except if [promoted] to a `static`. The [drop scope] of the temporary is usually the end of the enclosing statement.

### Implicit Borrows

Certain expressions will treat an expression as a place expression by implicitly
borrowing it. For example, it is possible to compare two unsized [slices][slice] for
equality directly, because the `==` operator implicitly borrows it's operands:

```rust
# let c = [1, 2, 3];
# let d = vec![1, 2, 3];
let a: &[i32];
let b: &[i32];
# a = &c;
# b = &d;
// ...
*a == *b;
// Equivalent form:
::std::cmp::PartialEq::eq(&*a, &*b);
```

Implicit borrows may be taken in the following expressions:

* Left operand in [method-call] expressions.
* Left operand in [field] expressions.
* Left operand in [call expressions].
* Left operand in [array indexing] expressions.
* Operand of the [dereference operator][deref] (`*`).
* Operands of [comparison].
* Left operands of the [compound assignment].

## Overloading Traits

Many of the following operators and expressions can also be overloaded for
other types using traits in `std::ops` or `std::cmp`. These traits also
exist in `core::ops` and `core::cmp` with the same names.

## 表达式属性

[Outer attributes][_OuterAttribute_] before an expression are allowed only in
a few specific cases:

* Before an expression used as a [statement].
* Elements of [array expressions], [tuple expressions], [call expressions],
  and tuple-style [struct] and [enum variant] expressions.
  <!--
    These were likely stabilized inadvertently.
    See https://github.com/rust-lang/rust/issues/32796 and
        https://github.com/rust-lang/rust/issues/15701
  -->
* The tail expression of [block expressions].
<!-- Keep list in sync with block-expr.md -->

They are never allowed before:
* [Range][_RangeExpression_] expressions.
* Binary operator expressions ([_ArithmeticOrLogicalExpression_],
  [_ComparisonExpression_], [_LazyBooleanExpression_], [_TypeCastExpression_],
  [_AssignmentExpression_], [_CompoundAssignmentExpression_]).


[block expressions]:    expressions/block-expr.md
[call expressions]:     expressions/call-expr.md
[enum variant]:         expressions/enum-variant-expr.md
[field]:                expressions/field-expr.md
[函数是更新]:             expressions/struct-expr.md#函数式更新句法
[`if let`]:             expressions/if-expr.md#if-let-expressions
[match]:                expressions/match-expr.md
[method-call]:          expressions/method-call-expr.md
[paths]:                expressions/path-expr.md
[struct]:               expressions/struct-expr.md
[tuple expressions]:    expressions/tuple-expr.md
[`while let`]:          expressions/loop-expr.md#predicate-pattern-loops

[array expressions]:    expressions/array-expr.md
[array indexing]:       expressions/array-expr.md#array-and-slice-indexing-expressions

[assign]:               expressions/operator-expr.md#assignment-expressions
[borrow]:               expressions/operator-expr.md#borrow-operators
[comparison]:           expressions/operator-expr.md#comparison-operators
[compound assignment]:  expressions/operator-expr.md#compound-assignment-expressions
[deref]:                expressions/operator-expr.md#the-dereference-operator

[destructors]:          destructors.md
[drop scope]:           destructors.md#drop-scopes

[`Box<T>`]:             ../std/boxed/struct.Box.html
[`Copy`]:               special-types-and-traits.md#copy
[`Drop`]:               special-types-and-traits.md#drop
[`Sized`]:              special-types-and-traits.md#sized
[implicit borrow]:      #implicit-borrows
[implicitly mutably borrowed]: #implicit-borrows
[interior mutability]:  interior-mutability.md
[let statement]:        statements.md#let-statements
[Mutable `static` items]: items/static-items.md#mutable-statics
[检验对象(scrutinee)]:            glossary.md#scrutinee
[promoted]:             destructors.md#constant-promotion
[slice]:                types/slice.md
[statement]:            statements.md
[static variables]:     items/static-items.md
[Temporary values]:     #temporaries
[Variables]:            variables.md

[_ArithmeticOrLogicalExpression_]: expressions/operator-expr.md#arithmetic-and-logical-binary-operators
[_ArrayExpression_]:              expressions/array-expr.md
[_AsyncBlockExpression_]:         expressions/block-expr.md#async-blocks
[_AwaitExpression_]:              expressions/await-expr.md
[_AssignmentExpression_]:         expressions/operator-expr.md#assignment-expressions
[_BlockExpression_]:              expressions/block-expr.md
[_BreakExpression_]:              expressions/loop-expr.md#break-expressions
[_CallExpression_]:               expressions/call-expr.md
[_ClosureExpression_]:            expressions/closure-expr.md
[_ComparisonExpression_]:         expressions/operator-expr.md#comparison-operators
[_CompoundAssignmentExpression_]: expressions/operator-expr.md#compound-assignment-expressions
[_ContinueExpression_]:           expressions/loop-expr.md#continue-expressions
[_EnumerationVariantExpression_]: expressions/enum-variant-expr.md
[_FieldExpression_]:              expressions/field-expr.md
[_GroupedExpression_]:            expressions/grouped-expr.md
[_IfExpression_]:                 expressions/if-expr.md#if-expressions
[_IfLetExpression_]:              expressions/if-expr.md#if-let-expressions
[_IndexExpression_]:              expressions/array-expr.md#array-and-slice-indexing-expressions
[_LazyBooleanExpression_]:        expressions/operator-expr.md#lazy-boolean-operators
[_LiteralExpression_]:            expressions/literal-expr.md
[_LoopExpression_]:               expressions/loop-expr.md
[_MacroInvocation_]:              macros.md#macro-invocation
[_MatchExpression_]:              expressions/match-expr.md
[_MethodCallExpression_]:         expressions/method-call-expr.md
[_OperatorExpression_]:           expressions/operator-expr.md
[_OuterAttribute_]:               attributes.md
[_PathExpression_]:               expressions/path-expr.md
[_RangeExpression_]:              expressions/range-expr.md
[_ReturnExpression_]:             expressions/return-expr.md
[_StructExpression_]:             expressions/struct-expr.md
[_TupleExpression_]:              expressions/tuple-expr.md
[_TupleIndexingExpression_]:      expressions/tuple-expr.md#tuple-indexing-expressions
[_TypeCastExpression_]:           expressions/operator-expr.md#type-cast-expressions
[_UnsafeBlockExpression_]:        expressions/block-expr.md#unsafe-blocks
