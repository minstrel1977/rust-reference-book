# 语句

>[statements.md](https://github.com/rust-lang/reference/blob/master/src/statements.md)\
>commit: 6b90080371ff44d0074a465945dfdb0de4b50774

> **<sup>句法</sup>**\
> _Statement_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `;`\
> &nbsp;&nbsp; | [_Item_]\
> &nbsp;&nbsp; | [_LetStatement_]\
> &nbsp;&nbsp; | [_ExpressionStatement_]\
> &nbsp;&nbsp; | [_MacroInvocationSemi_]

*语句*是[块(block)][^译者注]的一个组件，而块又是外部[表达式]或[函数]的组件。

Rust 有两种语句：[声明语句](#声明语句)和[表达式语句](#表达式语句)。

## 声明语句

*声明语句*是在封闭语句块中引入一个或多个*名称*的语句。声明的名称可以表示新变量或新的[数据项]。

这两种声明语句就是数据项声明和 let声明。

### 数据项声明

*数据项声明语句*的语法形式与[模块]中的[数据项声明][数据项]相同。在语句块中声明数据项会将该数据项的作用域限制为包含该语句的块，并且此数据项没有给定的[规范路径]，也无法（隐式）指定数据项的父子关系。例外的是由[实现]定义的关联项在外部范围内仍然是可访问的，只要数据项和 trait(如果适用的话)的可见性允许。除了这点儿区别外，它与在模块中声明数据项的意义是相同的。

数据项声明不会隐式捕获作用域内的组件，这些组件项包括函数的泛型参数、参数和局部变量。如下，`inner` 可能不能访问 `outer_var`。
<!-- There is no implicit capture of the containing function's generic parameters, parameters, and local variables. For example, `inner` may not access `outer_var`.TobeModify,这里为明确语义补充进来的“组件”可能不合适，回头可能会修改 -->

```rust
fn outer() {
  let outer_var = true;

  fn inner() { /* outer_var 的作用域不包括这里 */ }

  inner();
}
```

### `let`语句

> **<sup>句法</sup>**\
> _LetStatement_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> `let` [_Pattern_]
>     ( `:` [_Type_] )<sup>?</sup> (`=` [_Expression_] )<sup>?</sup> `;`

一个*`let`语句*引入了一组新的[变量]，由一个不可反驳型[模式]给出。模式后面有一个可选的类型标注，然后是一个初始化表达式。当没有给出类型标注时，编译器将自行推断类型，如果没有足够的信息来执行类型推断，则将触发编译器报错。从声明点到封闭块作用域的结束，变量声明引入的任何变量都是可见的。

## 表达式语句

> **<sup>句法</sup>**\
> _ExpressionStatement_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_ExpressionWithoutBlock_][expression] `;`\
> &nbsp;&nbsp; | [_ExpressionWithBlock_][expression] `;`<sup>?</sup>

*表达式语句*是对[表达式]求值并忽略其结果的语句。通常，表达式语句的目的是触发对其表达式求值的效果。

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

## 语句上的属性

语句可以有[外部属性]。在语句中有意义的属性是 [`cfg`] 和[lint检查类属性]。
Statements accept [outer attributes]. The attributes that have meaning on a statement are [`cfg`], and [lint检查类属性].

[^译者注]:我们口语中的说的代码块在本书原文书写为`block of code`,`block of code`的典型情况是`unsafe`块。

[块(block)]: expressions/block-expr.md
[表达式]: expressions.md
[函数]: items/functions.md
[数据项]: items.md
[模块]: items/modules.md
[规范路径]: paths.md#规范路径
[实现]: items/implementations.md
[变量]: variables.md
[外部属性]: attributes.md
[`cfg`]: conditional-compilation.md
[lint检查类属性]: attributes/diagnostics.md#lint检查类属性
[模式]: patterns.md
[_ExpressionStatement_]: #表达式语句
[_Expression_]: expressions.md
[_Item_]: items.md
[_LetStatement_]: #let-statements
[_MacroInvocationSemi_]: macros.md#宏调用
[_OuterAttribute_]: attributes.md
[_Pattern_]: patterns.md
[_Type_]: types.md
