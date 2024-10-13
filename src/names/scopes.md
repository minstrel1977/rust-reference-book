# Scopes
# 作用域

>[use-declarations.md](https://github.com/rust-lang/reference/blob/master/src/names/scopes.md)\
>commit: fcacb13cf9eccce3596e11a841bd7d8528a2921c \
>本章译文最后维护日期：2024-10-13

*作用域*是源文件中的区域，在这个区域中命名的[实体][entity]可以用该名称来引用。
下面的部分提供了关于作用域规则和行为的详细信息，这些规则和行为取决于实体的类型及其声明的位置。
[名称解析][name resolution]一章描述了如何将名称解析为实体的过程。
关于“删除作用域”的更多信息，可以在[析构函数][destructors]一章中找到。

## Item scopes
## 程序项作用域

在[模块][module]中直接声明的[程序项][items]的名称具有从模块开始到模块结束的作用域。这些程序项也是模块的成员，可以用以它们的模块名为前导的[路径][path]来引用。

声明为[语句][statement]的程序项的名称具有从该项语句所在的块的开始直到块的结束的作用域。

在同一模块（或块）内的同一[命名空间][namespace]中不可引入重复名称。
[`::*`式的导入][Asterisk glob imports] 在处理重复名称和遮蔽方面有特殊的行为，请参阅链接章节来了解更多详细信息。
模块中的程序项可能会遮蔽掉[预导入包](#prelude-scopes)中的程序项。

外部模块中的程序项名称在内部模块中不起作用。
[路径][path]可以用来指向另一个模块中的一个程序项。

### Associated item scopes
### 关联程序项的作用域

[关联项][Associated items]没有作用域，只能通过使用从它们关联的类型或 trait 名开始的[路径][path]来引用。
[方法][Methods]也可以通过[调用表达式][call expressions]引用。

与模块或块中的程序项类似，如果在 trait 或实现中引入一个程序项，而该项名与同一命名空间中的trait或impl中的另一个项名重复，则会出现错误。

## Pattern binding scopes
## 模式绑定的（符号）作用域

局部变量的[模式][pattern]绑定的作用域取决于使用它的位置：

* [`let`语句][`let` statement]这种绑定的作用域从 `let`语句之后一直到声明它的块的末尾。
* [函数参数][Function parameter]这种绑定的作用域在函数体中。
* [闭包参数][Closure parameter]这种绑定的作用域在闭包体中。
* [`for`] 和 [`while let`] 这种绑定在循环体中。
* [`if-let`] 这种绑定在后继块中。
* [`match` arms] 这种绑定在[匹配守卫][match guard]和匹配臂表达式中。

局部变量的作用域不会扩展到程序项的声明中。
<!-- Not entirely, see https://github.com/rust-lang/rust/issues/33118 -->

### Pattern binding shadowing
### 模式绑定的（符号） 遮蔽效应

允许模式绑定遮蔽作用域中的任何名称，但遮蔽以下程序项是错误的：

* [常量泛型参数][Const generic parameters]
* [静态项][Static items]
* [常量项][Const items]
* [结构体][structs]和[枚举][enums]的构造函数

下面的示例说明了本地绑定如何遮蔽已声明的程序项：
```rust
fn shadow_example() {
    // 由于作用域中还没有局部变量，因此将解析为函数。
    foo(); // prints `function`
    let foo = || println!("closure");
    fn foo() { println!("function"); }
    // 这解析为本地闭包，因为它遮蔽前面声明的程序项。
    foo(); // prints `closure`
}
```

## Generic parameter scopes
## 泛型参数的作用域

泛型参数在[_GenericParams_]列表中声明。
泛型参数的作用域在包含在声明它的程序项中。

所有参数都能在泛型参数列表中的作用域内有效，无论它们的声明顺序如何。
下面展示了一些参数在声明之前可以引用的示例：

```rust
// 'b在被定义之前就可以被使用
fn params_scope<'a: 'b, 'b>() {}

# trait SomeTrait<const Z: usize> {}
// 在声明常量N之前，N就能在 trait约束中使用
fn f<T: SomeTrait<N>, const N: usize>() {}
```
泛型参数在类型约束和 where子句的作用域内也算是域内有效的，例如：

```rust
# trait SomeTrait<'a, T> {}
// `SomeTrait` 的 <'a, U> 可以使用 `bounds_scope` 的泛型参数 'a 和 U。
fn bounds_scope<'a, T: SomeTrait<'a, U>, U>() {}

fn where_scope<'a, T, U>()
    where T: SomeTrait<'a, U>
{}
```

函数内声明的[程序项][items]不能从其外部作用域里引用泛型参数。

```rust,compile_fail
fn example<T>() {
    fn inner(x: T) {} // ERROR: can't use generic parameters from outer function
}
```

### Generic parameter shadowing
### 泛型参数的遮蔽效应

不能遮蔽泛型参数，但允许在函数中声明的程序项去遮蔽该函数的泛型形参名称。

```rust
fn example<'a, T, const N: usize>() {
    // 允许在函数中的声明的程序项在作用域中遮蔽上层泛型参数。
    fn inner_lifetime<'a>() {} // OK
    fn inner_type<T>() {} // OK
    fn inner_const<const N: usize>() {} // OK
}
```

```rust,compile_fail
trait SomeTrait<'a, T, const N: usize> {
    fn example_lifetime<'a>() {} // ERROR: 'a is already in use
    fn example_type<T>() {} // ERROR: T is already in use
    fn example_const<const N: usize>() {} // ERROR: N is already in use
    fn example_mixed<const T: usize>() {} // ERROR: T is already in use
}
```

### Lifetime scopes
### 生存期的作用域

生存期参数在 [_GenericParams_]列表和[高阶 trait约束][hrtb]中声明

`'static` 和[占位符生存期][placeholder lifetime] `'_` 拥有特殊的意义，不能被声明为参数。

#### Lifetime generic parameter scopes
#### 泛型生存期的作用域

[常量项][Constant]和[静态项][static]和[常量上下文][const contexts]只允许 `'static`生存期引用，因此它们中不能有其他生存期。
[关联常量][Associated consts]允许引用在其所在的trait或实现中声明的生存期。

#### Higher-ranked trait bound scopes
#### 高阶 trait约束的作用域

声明为[高阶 trait约束][hrtb]的生存期参数的作用域取决于使用它的场景。

* 作为 [_TypeBoundWhereClauseItem_] 所声明的生存期参数在类型和类型约束的作用域内有效。
* 作为 [_TraitBound_] 所声明的生存期参数在此类型约束路径所形成的作用域内有效。
* 作为 [_BareFunctionType_] 所声明的生存期参数在函数参数和返回类型的作用域内有效。

```rust
# trait Trait<'a>{}

fn where_clause<T>()
    // 'a 在类型和类型约束内都有效
    where for <'a> &'a T: Trait<'a>
{}

fn bound<T>()
    // 'a 只在约束内有效
    where T: for <'a> Trait<'a>
{}

# struct Example<'a> {
#     field: &'a u32
# }

// 'a 在函数参数和返回类型的作用域内有效
type FnExample = for<'a> fn(x: Example<'a>) -> Example<'a>;
```

#### Impl trait restrictions
#### Impl trait 时的限制

[Impl trait]类型只能引用在函数或实现上声明的生存期。

<!-- not able to demonstrate the scope error because the compiler panics
     https://github.com/rust-lang/rust/issues/67830
-->
```rust
# trait Trait1 {
#     type Item;
# }
# trait Trait2<'a> {}
#
# struct Example;
#
# impl Trait1 for Example {
#     type Item = Element;
# }
#
# struct Element;
# impl<'a> Trait2<'a> for Element {}
#
// 此处的 `impl Trait2` 不允许引用 'b，但允许引用 'a。
fn foo<'a>() -> impl for<'b> Trait1<Item = impl Trait2<'a> + use<'a>> {
    // ...
#    Example
}
```

## Loop label scopes
## 循环标签的作用域

[循环标签][Loop labels]可以由[循环表达式][loop expression]声明。
循环标签的作用域是从声明它的地方到循环表达式的末尾。
循环标签的作用域不会扩展到[程序项][items]、[闭包][closures]、[异步块][async blocks]、[常量实参][const arguments]、[常量上下文][const contexts]和定义 [for循环][`for` loop]的迭代器表达式。

```rust
'a: for n in 0..3 {
    if n % 2 == 0 {
        break 'a;
    }
    fn inner() {
        // 在这使用 'a 会报错。
        // break 'a;
    }
}

// The label is in scope for the expression of `while` loops.
'a: while break 'a {}         // Loop does not run.
'a: while let _ = break 'a {} // Loop does not run.

// The label is not in scope in the defining `for` loop:
'a: for outer in 0..5 {
    // 这将中断外循环，跳过内循环并停止外循环。T
    'a: for inner in { break 'a; 0..1 } {
        println!("{}", inner); // 不会执行到这里
    }
    println!("{}", outer); // 也不会执行到这里
}

```

循环标签可以遮蔽外部作用域中的同名的标签。
引用循环标签时引用的是最近定义的那个。

```rust
// 循环标签遮蔽效果示例。
'a: for outer in 0..5 {
    'a: for inner in 0..5 {
        // 这将终止内部循环，但外部循环将继续运行。
        break 'a;
    }
}
```

## Prelude scopes
## 预导入包的作用域

[预导入包][Preludes]将实体引入到模块的作用域内。
实体不是当前模块的成员，但在[名称解析][name resolution]期间隐式引入。
预导入包内实体名称可以被模块中的实体声明所遮蔽。

预导入包是分层的，因此如果它们中包含了同名的实体，则后相互遮蔽。
预导入包遮蔽其他预导入包的顺序如下，其中较早导入的程序项可以遮蔽较晚导入的：

1. [Extern prelude]
2. [Tool prelude]
3. [`macro_use` prelude]
4. [Standard library prelude]
5. [Language prelude]

## `macro_rules` scopes
## 声明宏的作用域

`macro_rules`宏的作用域在[macros By Example]一章中详述。
声明宏的行为取决于 [`macro_use`] 和 [`macro_export`]属性的使用。

## Derive macro helper attributes
## 派生宏辅助属性

[派生宏辅助属性][Derive macro helper attributes]在应用了相应[`derive`属性][`Derive` attribute]的程序项的作用域内有效。
这种作用域从 `derive`属性之后扩展到该程序项的末尾。<！--注：严格来说不完全正确，具体请参阅https://github.com/rust-lang/rust/issues/79202 -->
辅助对象属性在作用域内会遮蔽同名的其他属性。

## `Self` scope
## `Self`作用域

尽管 [`Self`] 是一个具有特殊含义的关键字，但它与名称解析的交互方式类似于普通名称。

[struct]、[enum]、[union]、[trait]或[implementation]的定义中的隐式 `Self`类型被用类似于[泛型参数](#generic-parameter-scopes)的方式来处理，并以与泛型类型参数相同的方式来生效作用域。

[implementation]中的隐式`Self`值构造函数在实现的主体代码（实现的[关联项][associated items]）的作用域内有效。

```rust
// Self type within struct definition.
struct Recursive {
    f1: Option<Box<Self>>
}

// Self type within generic parameters.
struct SelfGeneric<T: Into<Self>>(T);

// Self value constructor within an implementation.
struct ImplExample();
impl ImplExample {
    fn example() -> Self { // Self type
        Self() // Self value constructor
    }
}
```

[_BareFunctionType_]: ../types/function-pointer.md
[_GenericParams_]: ../items/generics.md
[_TraitBound_]: ../trait-bounds.md
[_TypeBoundWhereClauseItem_]: ../items/generics.md
[`derive` attribute]: ../attributes/derive.md
[`for` loop]: ../expressions/loop-expr.md#iterator-loops
[`for`]: ../expressions/loop-expr.md#iterator-loops
[`if let`]: ../expressions/if-expr.md#if-let-expressions
[`let` statement]: ../statements.md#let-statements
[`macro_export`]: ../macros-by-example.md#path-based-scope
[`macro_use` prelude]: preludes.md#macro_use-prelude
[`macro_use`]: ../macros-by-example.md#the-macro_use-attribute
[`match` arms]: ../expressions/match-expr.md
[`Self`]: ../paths.md#self-1
[`while let`]: ../expressions/loop-expr.md#predicate-pattern-loops
[Associated consts]: ../items/associated-items.md#associated-constants
[associated items]: ../items/associated-items.md
[Asterisk glob imports]: ../items/use-declarations.md
[async blocks]: ../expressions/block-expr.md#async-blocks
[call expressions]: ../expressions/call-expr.md
[Closure parameter]: ../expressions/closure-expr.md
[closures]: ../expressions/closure-expr.md
[const arguments]: ../items/generics.md#const-generics
[const contexts]: ../const_eval.md#const-context
[Const generic parameters]: ../items/generics.md#const-generics
[Const items]: ../items/constant-items.md
[Constant]: ../items/constant-items.md
[Derive macro helper attributes]: ../procedural-macros.md#derive-macro-helper-attributes
[destructors]: ../destructors.md
[entity]: ../names.md
[enum]: ../items/enumerations.mdr
[enums]: ../items/enumerations.md
[Extern prelude]: preludes.md#extern-prelude
[Function parameter]: ../items/functions.md#function-parameters
[hrtb]: ../trait-bounds.md#higher-ranked-trait-bounds
[Impl trait]: ../types/impl-trait.md
[implementation]: ../items/implementations.md
[items]: ../items.md
[Language prelude]: preludes.md#language-prelude
[loop expression]: ../expressions/loop-expr.md
[Loop labels]: ../expressions/loop-expr.md#loop-labels
[Macros By Example]: ../macros-by-example.md
[match guard]: ../expressions/match-expr.md#match-guards
[methods]: ../items/associated-items.md#methods
[module]: ../items/modules.md
[name resolution]: name-resolution.md
[namespace]: namespaces.md
[path]: ../paths.md
[pattern]: ../patterns.md
[placeholder lifetime]: ../lifetime-elision.md
[preludes]: preludes.md
[Standard library prelude]: preludes.md#standard-library-prelude
[statement]: ../statements.md
[Static items]: ../items/static-items.md
[static]: ../items/static-items.md
[struct]: ../items/structs.md
[structs]: ../items/structs.md
[Tool prelude]: preludes.md#tool-prelude
[trait]: ../items/traits.md
[union]: ../items/unions.md