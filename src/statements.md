# Statements
# 语句

>[statements.md](https://github.com/rust-lang/reference/blob/master/src/statements.md)\
>commit: 6b90080371ff44d0074a465945dfdb0de4b50774 \
>本译文最后维护日期：2020-10-24

> **<sup>句法</sup>**\
> _Statement_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `;`\
> &nbsp;&nbsp; | [_Item_]\
> &nbsp;&nbsp; | [_LetStatement_]\
> &nbsp;&nbsp; | [_ExpressionStatement_]\
> &nbsp;&nbsp; | [_MacroInvocationSemi_]

*语句*是[块(block)][block][^译者注]的一个组件，相应地，块又是外层[表达式][expression]或[函数][function]的组件。

Rust 有两种语句：[声明式语句(declaration statements)](#declaration-statements)和[表达式语句(expression statements)](#expression-statements)。

## Declaration statements
## 声明式语句

*声明式语句*是在封闭语句块中引入一个或多个*名称*的语句。声明的名称可以表示新变量或新的[数据项][item]。

这两种声明式语句就是数据项声明和 let声明。

### Item declarations
### 数据项声明

*数据项声明语句*的句法形式与[模块][module]中的[数据项声明][item]的句法形式相同。在语句块中声明的数据项会将其作用域限制为包含该语句的块。这类数据项以及其子项都没有给定的[规范路径][canonical path]。例外的是，只要数据项和 trait(如果有的话)的可见性允许，由[实现][implementations]定义的关联项在外层作用域内仍然是可访问的。除了这些区别外，它与在模块中声明数据项的意义也相同的。

数据项声明不会隐式捕获包含它的函数的泛型参数、参数和局部变量。如下，`inner` 不能访问 `outer_var`。

`outer_var`.
```rust
fn outer() {
  let outer_var = true;

  fn inner() { /* outer_var 的作用域不包括这里 */ }

  inner();
}
```

### `let` statements
### `let`语句

> **<sup>句法</sup>**\
> _LetStatement_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> `let` [_Pattern_]
>     ( `:` [_Type_] )<sup>?</sup> (`=` [_Expression_] )<sup>?</sup> `;`

*`let`语句*通过一个不可反驳型[模式][pattern]引入了一组新的[变量][variables]，变量由该模式给定。模式后面有一个可选的类型标注(annotation)，再后面是一个初始化表达式。当没有给出类型标注时，编译器将自行推断类型，如果没有足够的信息来执行类型推断，则将触发编译器报错。由变量声明引入的任何变量从声明开始直到封闭块作用域结束都是可见的。

## Expression statements
## 表达式语句

> **<sup>句法</sup>**\
> _ExpressionStatement_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_ExpressionWithoutBlock_][expression] `;`\
> &nbsp;&nbsp; | [_ExpressionWithBlock_][expression] `;`<sup>?</sup>

*表达式语句*是对[表达式][expression]求值并忽略其结果的语句。通常，表达式语句的目的是触发（其包含的表达式的）求值效果的表达。

仅由[块表达式][block]或控制流表达式组成的表达式，如果在允许语句的上下文中使用，则可以省略后面的分号。这有可能会导致解析歧义，因为它可以被解析为独立语句，也可以被解析为另一个表达式的一部分；在下例中，它被解析为一个语句。[_ExpressionWithBlock_][expression] 形式的表达式用作语句时，其类型必须是 单元类型（`()`）。

```rust
# let mut v = vec![1, 2, 3];
v.pop();          // 忽略从 pop 返回的元素
if v.is_empty() {
    v.push(5);
} else {
    v.remove(0);
}                 // 分号可以省略。
[1];              // 单独的表达式语句，而不是索引表达式。
```

当省略后面的分号时，结果必须是 `()` 类型。

```rust
// bad: 下面块的类型是i32，而不是 `()` 
// Error: 预期表达式语句的返回值是 `()` 
// if true {
//   1
// }

// good: 下面块的类型是i32，（加`;`后的语句的返回值就是 `()`了）
if true {
  1
} else {
  2
};
```

## Attributes on Statements
## 语句上的属性

语句可以有[外部属性][outer attributes]。在语句中有意义的属性是 [`cfg`] 和 [lint检查类属性][the lint check attributes]。

[^译者注]: 本书原文还有 `block of code` 的写法，这种有些类似于我们口语中说的那种任意的代码段的“代码块”。原文中 `block of code` 的典型情况是非安全(`unsafe`)块。

[block]: expressions/block-expr.md
[expression]: expressions.md
[function]: items/functions.md
[item]: items.md
[module]: items/modules.md
[canonical path]: paths.md#canonical-paths
[implementations]: items/implementations.md
[variables]: variables.md
[outer attributes]: attributes.md
[`cfg`]: conditional-compilation.md
[the lint check attributes]: attributes/diagnostics.md#lint-check-attributes
[pattern]: patterns.md
[_ExpressionStatement_]: #expression-statements
[_Expression_]: expressions.md
[_Item_]: items.md
[_LetStatement_]: #let-statements
[_MacroInvocationSemi_]: macros.md#macro-invocation
[_OuterAttribute_]: attributes.md
[_Pattern_]: patterns.md
[_Type_]: types.md

<!-- 2020-10-25 -->
<!-- checked -->
