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

当位置表达式在值表达式上下文中求值，或在模式中被值绑定时，这表示求出的值会*保存进*（held in）当前表达式代表的内存地址。如果该值的类型实现了 [`Copy`]，那么该值将被从原来的位置表达式（也可以理解为原来的内存位置）中复制一份过来。如果该值的类型没有实现 [`Copy`]，但实现了 [`Sized`]，那么就可以把该值从原来的位置表达式里移出（move out）（到新位置表达式中）。移出对位置表达式也有要求，具体如下的位置表达式里的值才可以被移出：<!-- When a place expression is evaluated in a value expression context, or is bound by value in a pattern, it denotes the value held _in_ that memory location. If the type of that value implements [`Copy`], then the value will be copied. In the remaining situations if that type is [`Sized`], then it may be possible to move the value. Only the following place expressions may be moved out of: 这里直译后看不懂，意译又怕理解错误，只能先打个标记 TobeModif-->

* [变量]（译者注：位置表达式的一种）当前未被借用。
* [临时值](#临时位置)。
* 可以移出且没实现 [`Drop`] 的位置表达式的字段。<!-- [Fields][field] of a place expression which can be moved out of and doesn't implement [`Drop`]. TobeModify-->
* 对可移出且类型为 [`Box<T>`] 的表达式[解引用][deref]的结果。<!-- The result of [dereferencing][deref] an expression with type [`Box<T>`] and that can also be moved out of. TobeModify-->

移出被作为局部变量的位置表达式里的值后，原来的地址将被去初始化（deinitialized），并且该地址在重新初始化之前无法再次读取。除以上列出的情况为外，尝试在值表达式上下文中使用位置表达式都是错误的。

### 可变性

对于表[示分配][assign]、可变[借用][borrow]、[隐式可变借用]或绑定到包含 `ref mut` 的模式的位置表达式必须是[可变的]。我们称这些为*可变位置表达式*。与之相比，其他位置表达式称为*不可变位置表达式*。

下面的表达式可以是可变位置表达式上下文：

* 当前未出借的可变的[变量]。
* [可变 `static`数据项]。
* [临时值]。
* 求值结果是可变位置表达式上下文的[字段][field]。
* 对 `*mut T` 指针的[解引用][deref]。
* 对类型为 `&mut T` 的变量或变量的字段的解引用。注意：这是下一条规则的例外情况。
* 实现 `DerefMut` 的类型的解引用，这就要求被解引用的值是一个可变位置表达式上下文
* 对于实现 `IndexMut` 的类型的[数组索引]，它将在可变位置表达式上下文中计算被索引到的值，而不是索引本身。

### 临时位置

在大多数位置表达式上下文中使用值表达式时，会创建一个临时的未命名内存位置，并将该值初始化到该内存位置，而表达式将求值结果在存放到该位置。也有例外，就把此表达式[提升]为 `static`。（译者注：这种情况下表达式将直接在编译时就求值了，存储地址会根据编译器要求任意放置，甚至多个位置放置）。临时位置的[销毁点][drop scope]通常在其封闭语句的结尾处。

### 隐式借用

某些表达式可通过隐式借用表达式来将其视为位置表达式。例如，可以直接比较两个[切片][slice]是否相等，因为 `==` 运算符隐式借用了它的操作数：

```rust
# let c = [1, 2, 3];
# let d = vec![1, 2, 3];
let a: &[i32];
let b: &[i32];
# a = &c;
# b = &d;
// ...
*a == *b; //译者注：&[i32] 解引用后是一个动态尺寸类型，理论上两个动态尺寸类型上无法比较大小的，但这里因为隐式借用此成为可能
// 等价于下面的形式:
::std::cmp::PartialEq::eq(&*a, &*b);
```

隐式借用可采用以下表达式：

* [方法调用][method-call]表达式中的左操作数。
* [字段][field]表达式中的左操作数。
* [调用表达式][call expressions]中的左操作数。
* [数组索引][array indexing]表达式中的左操作数。
* [解引用操作符][deref]（`*`）的操作数。
* [比较运算][comparison]的操作数。
* [复合赋值][compound assignment]的左操作数。

## 重载

本节之后的许多操作符和表达式都可以通过 `std::ops` 或 `std::cmp` 中的 trait 被其他类型重载。这些 trait 也存在于同名的 `core::ops` 和 `core::cmp` 中。

## 表达式属性

只有在少数特定情况下，才允许在表达式之前使用[外部属性][_OuterAttribute_]：

* 在被用作[语句]的表达式之前。
* [数组表达式]、[元组表达式]、[调用表达式]和[元组结构体]和[枚举变体]表达式这些中的元素。
  <!--
    These were likely stabilized inadvertently.
    See https://github.com/rust-lang/rust/issues/32796 and
        https://github.com/rust-lang/rust/issues/15701
  -->
* [块表达式]的尾部表达式.
<!-- Keep list in sync with block-expr.md -->

在下面情形之前是不允许的：
* [范围][_RangeExpression_]表达式。
* 二元运算符表达式([_ArithmeticOrLogicalExpression_]、[_ComparisonExpression_]、[_LazyBooleanExpression_]、[_TypeCastExpression_]、[_AssignmentExpression_]、[_CompoundAssignmentExpression_])。


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
