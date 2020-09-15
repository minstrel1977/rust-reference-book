# 函数

>[functions.md](https://github.com/rust-lang/reference/blob/master/src/items/functions.md)\
>commit e6e7e657e2aae2cc474d1324d3f4508bd7b1d08c

> **<sup>句法</sup>**\
> _Function_ :\
> &nbsp;&nbsp; _FunctionQualifiers_ `fn` [IDENTIFIER]&nbsp;[_Generics_]<sup>?</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp; `(` _FunctionParameters_<sup>?</sup> `)`\
> &nbsp;&nbsp; &nbsp;&nbsp; _FunctionReturnType_<sup>?</sup> [_WhereClause_]<sup>?</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp; [_BlockExpression_]
>
> _FunctionQualifiers_ :\
> &nbsp;&nbsp; _AsyncConstQualifiers_<sup>?</sup> `unsafe`<sup>?</sup> (`extern` _Abi_<sup>?</sup>)<sup>?</sup>
>
> _AsyncConstQualifiers_ :\
> &nbsp;&nbsp; `async` | `const`
>
> _Abi_ :\
> &nbsp;&nbsp; [STRING_LITERAL] | [RAW_STRING_LITERAL]
>
> _FunctionParameters_ :\
> &nbsp;&nbsp; _FunctionParam_ (`,` _FunctionParam_)<sup>\*</sup> `,`<sup>?</sup>
>
> _FunctionParam_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> [_Pattern_] `:` [_Type_]
>
> _FunctionReturnType_ :\
> &nbsp;&nbsp; `->` [_Type_]

函数由一个[代码块]以及一个名称和一组参数组成。除了名字，所有这些都是可选的。函数用关键字  `fn` 声明。函数可以声明一组*输入*[*变量*][variables]作为参数，调用者通过它向函数传递参数，函数完成后将带有[*类型*][type]的*输出*值返回给调用者。

当*函数*被引用时，会产生相应的零尺寸[*函数类型(function item type)*]的一等*值*，当调用该值时，它的计算结果是对该函数的直接调用。

举个简单的函数例子:
```rust
fn answer_to_life_the_universe_and_everything() -> i32 {
    return 42;
}
```

和 `let` 绑定一样，函数参数是不可反驳的[模式]，所以任何在 let 绑定中有效的模式作为参数也是有效的:

```rust
fn first((value, _): (i32, i32)) -> i32 { value }
```

函数的代码块在概念上被包装在一个块中，该块绑定参数模式，然后返回函数块的值。这意味着块的*尾部表达式*如果能进行求值运算，最终会将该运算后的值返回给调用者。通常，函数体中显式的*返回表达式*会缩短隐式返回的时间。

例如，上面例子里函数的行为就像下面被改写的这样:

<!-- ignore: example expansion -->
```rust,ignore
// argument_0 是调用者真正传过来的第一个参数
let (value, _) = argument_0;
return {
    value
};
```

## 泛型函数

*泛型函数*允许在其签名中出现一个或多个*参数化类型*。每个类型参数必须在函数名后面的尖括号和逗号分隔的列表中显式声明。

```rust
// foo 建立在 A 和 B 基础上的泛型

fn foo<A, B>(x: A, y: B) {
# }
```

在函数签名和主体内部，类型参数的名称可以用作类型名称。可以为类型参数指定 [trait] 约束，以允许对该类型的值调用具有该 trait 的方法。这是使用 `where` 句法指定的:

```rust
# use std::fmt::Debug;
fn foo<T>(x: T) where T: Debug {
# }
```

当引用泛型函数时，它的类型将基于引用的上下文被实例化。例如，这里调用 `foo` 函数:

```rust
use std::fmt::Debug;

fn foo<T>(x: &[T]) where T: Debug {
    // 省略细节
}

foo(&[1, 2]);
```

将用 `i32` 实例化类型参数 `T`。

类型参数也可以在函数名后面的[路径]组件中显式地提供。如果没有足够的上下文来确定类型参数，那么这可能是必要的。例如：`mem::size_of::<u32>() == 4`。

## 外来函数限定符

`extern` 函数限定符允许提供可以通过特定 ABI 调用的函数*定义*：
<!-- ignore: fake ABI -->
```rust,ignore
extern "ABI" fn foo() { /* ... */ }
```

这些通常与[外部块]数据项一起使用，这些数据项提供函数*声明*，可以用来调用函数而不提供它们的*定义*:

<!-- ignore: fake ABI -->
```rust,ignore
extern "ABI" {
  fn foo(); /* no body */
}
unsafe { foo() }
```

当函数的句法结构 `FunctionQualifiers` 中的 `"extern" Abi?*` 被省略时，ABI `"Rust"` 会被赋值。例如:

```rust
fn foo() {}
```

等价于：

```rust
extern "Rust" fn foo() {}
```

Rust 中的函数可以被其他编程语言调用，例如使用不同于 Rust 的 ABI 提供了可以从其他编程语言(如 C)调用的函数:

```rust
// 使用 "C" ABI 声明一个函数
extern "C" fn new_i32() -> i32 { 0 }

// 使用 "stdcall" ABI声明一个函数
# #[cfg(target_arch = "x86_64")]
extern "stdcall" fn new_i32_stdcall() -> i32 { 0 }
```

与[外部块]一样，当使用关键字 `extern` 而省略 `"ABI` 时，ABI 默认使用 `"C"`。也就是说这个：

```rust
extern fn new_i32() -> i32 { 0 }
let fptr: extern fn() -> i32 = new_i32;
```

等价于：

```rust
extern "C" fn new_i32() -> i32 { 0 }
let fptr: extern "C" fn() -> i32 = new_i32;
```

非 `"Rust"` 的 ABI 函数不支持与 Rust 函数完全相同的 unwind 方式。因此展开碰到带有这类 ABI 的函数的结束时会导致进程终止。
<!--TobeModify: Functions with an ABI that differs from `"Rust"` do not support unwinding in the exact same way that Rust does. Therefore, unwinding past the end of functions with such ABIs causes the process to abort.-->

> **注意**: `rustc` 实现背后的 LLVM 会通过执行一个非法指令来中止进程。

## 常量函数

使用关键字 `const` 限定的函数是常量函数，与[元组结构体]和[元祖变体]构造函数一样。*常量函数*可以在[常量上下文]中调用。当从常量上下文中调用这类函数时，编译器会在编译时解释该函数。这种解释发生在(Rust编译器为)编译目标(构建的模拟)环境中，而不是在当前主机环境中。因此，如果您是针对一个 `32` 位系统进行编译，那么 `usize` 就是 `32` 位，这与您在一个 `64` 位还是在一个 `32` 位系统上进行编译无关。

如果在[常量上下文]之外调用常量函数，那么它与任何其他函数没有区别。你可以自由地用常量函数做任何你可以用普通函数做的任何事情。

常量函数有各种限制以确保其可以在编译时对其求值。例如，不可以将随机数生成器编写为常量函数。在编译时调用常量函数将始终产生与运行时调用它相同的结果，即使多次调用也是如此。这个规则有一个例外：如果您在极端情况下执行复杂的浮点运算，那么您可能得到（非常轻微）不同的结果。建议不要使数组长度和枚举判别式依赖于浮点计算。

允许出现在常量函数中的数据结构的详尽列表：
<!--Exhaustive list of permitted structures in const functions: TobeModify-->

> **注意**: 这个列表比你可以用常规常量写的东西更具限制性

* 类型参数中参数只能有以下类型的 [trait 约束]：
<!--* Type parameters where the parameters only have any [trait bounds] of the following kind: TobeModify-->
    * 生命周期
    * `Sized` 或 [`?Sized`]

    这意味着 `<T: 'a + ?Sized>`、`<T: 'b + Sized>` 和 `<T>` 都是可以的。
    
    此规则也适用于包含常量方法的 *impl 块*的类型参数。
    
    此规则不适用于元组结构体和元组变体构造函数。    

* 整数上的算术和比较运算符
* 除 `&&` 和 `||` 之外的所有布尔运算符，这俩是因为它们的短路运算而被禁止。
* 任何类型的聚合构造函数（数组、`struct`、`enum`、元组，…）
* 对其他*安全*常量函数的调用（无论是通过函数调用还是通过方法调用）
* 数组和切片上的索引表达式
* 对结构体和元组的字段的访问
* 从常量数据项（但不能是静态数据项，甚至不能引用静态数据项）中读取数据
* `&` 和 `*`（仅解引用引用，原始指针不行）
* 除裸指针向整数转换外的类型转换
* `unsafe` 块和 `const unsafe fn` 可以，但代码体/代码块只能执行以下非安全操作：
    * 调用非安全常量函数

## 异步函数 （async functions）

函数可以限定为 `async`，这也可以与 `unsafe` 限定符结合使用：

```rust,edition2018
async fn regular_example() { }
async unsafe fn unsafe_example() { }
```

异步函数在调用时不起作用：相反，它们将参数捕获进 future。当该函数被轮询时，该 future 将执行该函数的函数体。

一个异步函数大致相当于返回一个以 [`async move` 代码块][async-blocks]为代码体的 [`impl Future`] 的函数：

```rust,edition2018
// 源代码
async fn example(x: &str) -> usize {
    x.len()
}
```

大致等价于：

```rust,edition2018
# use std::future::Future;
// 脱糖后的
fn example<'a>(x: &'a str) -> impl Future<Output = usize> + 'a {
    async move { x.len() }
}
```

实际的脱糖过程相当复杂：

- 假定脱糖过程中的返回类型捕获 *`async fn` 声明*里的所有生命周期。这可以在上面的脱糖示例中看到：（我们）显式地给它添加了一个最小生存期 `'a`，因此捕捉到 `'a`。
- 代码体中的 [`async move` 块][async blocks]捕获所有函数参数，包括未使用或绑定到 `_` 模式的参数。这可以确保函数参数的销毁顺序与函数非异步时的顺序相同，除了这些销毁动作需要在返回的 future 完全执行(fully awaited)完成后才会发生。

有关异步效果的详细信息，请参见[`async` 块][async-blocks]。

[async-blocks]: ../expressions/block-expr.md#async-blocks
[`impl Future`]: ../types/impl-trait.md

> **版本差异**: 异步函数只能在 Rust 2018 版中才有效。

### `async` 和 `unsafe` 的结合

声明一个既异步又非安全的函数是合法的。调用这样的函数是非安全的，并且（像任何异步函数一样）会返回一个 future。这个 future 只是一个普通的 future，因此“ await ”它不需要一个 `unsafe` 的上下文：

```rust,edition2018
// 等待这个返回的 future 相当于解引用 `x`.
//
// 安全条件: 指导返回的 future 执行完成，`x` 必须能被安全解引用
async unsafe fn unsafe_example(x: *const i32) -> i32 {
  *x
}

async fn safe_example() {
    // 起初调用上面这个函数时需要一个 `unsafe` 块：
    let p = 22;
    let future = unsafe { unsafe_example(&p) };

    // 但是这里`unsafe` 块就没必要了，这里能正常读到 `p`:
    let q = future.await;
}
```

请注意，此行为是对返回 `impl Future` 的函数进行脱糖处理的结果——在本例中，我们设计的函数是一个 `unsafe` 函数，但返回值保持不变。

非安全在异步函数上的使用方式与它在其他函数上的使用方式完全相同：它表示该函数的调用者需要遵循一些额外的协议来确保调用执行的可靠性。与任何其他不安全函数一样，这些条件可能会超出初始调用本身——例如，在上面的代码片段中， `unsafe_example` 函数将指针 `x` 作为参数，然后（在执行 await 时）解引用了对该指针的引用。这意味着在 future 完成执行之前， `x` 必须是有效的，调用者有责任确保这一点。#

## Attributes on functions

[Outer attributes][attributes] are allowed on functions. [Inner
attributes][attributes] are allowed directly after the `{` inside its [block].

This example shows an inner attribute on a function. The function will only be
available while running tests.

```
fn test_only() {
    #![test]
}
```

> Note: Except for lints, it is idiomatic to only use outer attributes on
> function items.

The attributes that have meaning on a function are [`cfg`], [`cfg_attr`], [`deprecated`],
[`doc`], [`export_name`], [`link_section`], [`no_mangle`], [the lint check
attributes], [`must_use`], [the procedural macro attributes], [the testing
attributes], and [the optimization hint attributes]. Functions also accept
attributes macros.

## Attributes on function parameters

[Outer attributes][attributes] are allowed on function parameters and the
permitted [built-in attributes] are restricted to `cfg`, `cfg_attr`, `allow`,
`warn`, `deny`, and `forbid`.

```rust
fn len(
    #[cfg(windows)] slice: &[u16],
    #[cfg(not(windows))] slice: &[u8],
) -> usize {
    slice.len()
}
```

Inert helper attributes used by procedural macro attributes applied to items are also
allowed but be careful to not include these inert attributes in your final `TokenStream`.

For example, the following code defines an inert `some_inert_attribute` attribute that
is not formally defined anywhere and the `some_proc_macro_attribute` procedural macro is
responsible for detecting its presence and removing it from the output token stream.

<!-- ignore: requires proc macro -->
```rust,ignore
#[some_proc_macro_attribute]
fn foo_oof(#[some_inert_attribute] arg: u8) {
}
```

[IDENTIFIER]: ../identifiers.md
[RAW_STRING_LITERAL]: ../tokens.md#raw-string-literals
[STRING_LITERAL]: ../tokens.md#string-literals
[_BlockExpression_]: ../expressions/block-expr.md
[_Generics_]: generics.md
[_Pattern_]: ../patterns.md
[_Type_]: ../types.md#type-expressions
[_WhereClause_]: generics.md#where-clauses
[_OuterAttribute_]: ../attributes.md
[const context]: ../const_eval.md#const-context
[tuple struct]: structs.md
[tuple variant]: enumerations.md
[external block]: external-blocks.md
[path]: ../paths.md
[block]: ../expressions/block-expr.md
[variables]: ../variables.md
[type]: ../types.md#type-expressions
[*函数类型(function item type)*]: ../types/function-item.md
[Trait]: traits.md
[attributes]: ../attributes.md
[`cfg`]: ../conditional-compilation.md#the-cfg-attribute
[`cfg_attr`]: ../conditional-compilation.md#the-cfg_attr-attribute
[the lint check attributes]: ../attributes/diagnostics.md#lint-check-attributes
[the procedural macro attributes]: ../procedural-macros.md
[the testing attributes]: ../attributes/testing.md
[the optimization hint attributes]: ../attributes/codegen.md#optimization-hints
[`deprecated`]: ../attributes/diagnostics.md#the-deprecated-attribute
[`doc`]: ../../rustdoc/the-doc-attribute.html
[`must_use`]: ../attributes/diagnostics.md#the-must_use-attribute
[patterns]: ../patterns.md
[`?Sized`]: ../trait-bounds.md#sized
[trait bounds]: ../trait-bounds.md
[`export_name`]: ../abi.md#the-export_name-attribute
[`link_section`]: ../abi.md#the-link_section-attribute
[`no_mangle`]: ../abi.md#the-no_mangle-attribute
[external_block_abi]: external-blocks.md#abi
[built-in attributes]: ../attributes.html#built-in-attributes-index