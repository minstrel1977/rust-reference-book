# 术语表

>[glossary.md](https://github.com/rust-lang/reference/blob/master/src/glossary.md)\
>commit eadbdafec0c4e302c2e564a773fe5bff60ddac1d

### Abstract syntax tree
抽象语法树

当编译器编译程序时，抽象语法树(AST)是程序结构的中间表示。
<!-- An ‘abstract syntax tree’, or ‘AST’, is an intermediate representation of the structure of the program when the compiler is compiling it. -->

### Alignment
对齐

值的对齐方式指定值的首选起始地址。对齐的值总是2的次方数。对值的引用必须对齐（译者理解：值的对齐决定值的起始存储地址，那引用该值就相当于取这个值的存储首地址，正好引用中带有值的类型，那就顺便做一次简单的内存校验）。[更多][alignment]。
<!-- The alignment of a value specifies what addresses values are preferred to start at. Always a power of two. References to a value must be aligned. [More][alignment]. -->

### Arity
元数

元数是指函数或运算符接受的参数个数。例如，`f(2, 3)` 和 `g(4, 6)` 的元数为2，而 `h(8, 2, 6)` 的元数为3。 `!` 运算符的元数为1。
<!-- Arity refers to the number of arguments a function or operator takes. For some examples, `f(2, 3)` and `g(4, 6)` have arity 2, while `h(8, 2, 6)` has arity 3. The `!` operator has arity 1. -->

### Array
数组

数组，有时也称为固定大小数组或内联数组，是描述元素集合的值，每个元素都由可由程序在运行时给出的索引选择。数组占用内存的一个连续区域。
<!-- An array, sometimes also called a fixed-size array or an inline array, is a value describing a collection of elements, each selected by an index that can be computed at run time by the program. It occupies a contiguous region of memory. -->

### Associated item
关联数据项

关联数据项是与另一个数据项关联的数据项。关联数据项在[实现]中定义，并在[traits]中声明。关联数据项只能关联函数、常量和类型别名。它与[自由数据项]形成对比。
<!-- An associated item is an item that is associated with another item. Associated items are defined in [implementations] and declared in [traits]. Only functions, constants, and type aliases can be associated. Contrast to a [free item]. -->

### Blanket implementation
blanket implementation

指[无覆盖类型](#uncovered-type)上的任何实现。`impl<T> Foo for T`、`impl<T> Bar<T> for T`、`impl<T> Bar<Vec<T>> for T`、 和 `impl<T> Bar<T> for Vec<T>` 被认为是 blanket implementation。但是，`impl<T> Bar<Vec<T>> for Vec<T>` 不被认为是，因为这个 `impl` 中所有的 `T` 的实例都被 `Vec` 覆盖。
<!-- Any implementation where a type appears [uncovered](#uncovered-type). `impl<T> Foo for T`, `impl<T> Bar<T> for T`, `impl<T> Bar<Vec<T>> for T`, and `impl<T> Bar<T> for Vec<T>` are considered blanket impls. However, `impl<T> Bar<Vec<T>> for Vec<T>` is not a blanket impl, as all instances of `T` which appear in this `impl` are covered by `Vec`. -->

### Bound
约束

约束是对类型或 trait 的限制。例如，如果在函数接受的参数上设置了约束，则传递给该函数的（值的）类型必须遵守该约束。
<!-- Bounds are constraints on a type or trait. For example, if a bound is placed on the argument a function takes, types passed to that function must abide by that constraint. -->

### Combinator
组合子

组合子是高阶函数，它的参数全是函数或之前定义的组合子。组合子利用这些函数或组合子返回的结果作为入参进行进一步的逻辑计算和输出。组合子多用于以模块化的方式管理控制流。
<!-- Combinators are higher-order functions that apply only functions and earlier defined combinators to provide a result from its arguments. They can be used to manage control flow in a modular fashion. -->

### Dispatch
分发

分发是一种机制，用于确定涉及到多态性时实际运行的是哪个版本的代码。分发的两种主要形式是静态分发和动态分发。虽然Rust支持静态分发，但它也通过一种称为 trait对象的机制支持动态分发
<!-- Dispatch is the mechanism to determine which specific version of code is actually run when it involves polymorphism. Two major forms of dispatch are static dispatch and dynamic dispatch. While Rust favors static dispatch, it also supports dynamic dispatch through a mechanism called ‘trait objects’. -->

### Dynamically sized type
动态类型

动态尺寸类型(DST)是一种没有静态已知大小或对齐方式的类型
<!-- A dynamically sized type (DST) is a type without a statically known size or alignment. -->

### Expression
表达式

表达式是值、常量、变量、运算符和函数的组合，计算结果为单个值，有或没有副作用都有可能。
<!-- An expression is a combination of values, constants, variables, operators and functions that evaluate to a single value, with or without side-effects. -->
比如，`2 + (3 * 4)` 是一个返回值为14的表达式。
<!-- For example, `2 + (3 * 4)` is an expression that returns the value 14. -->

### Free item
自有数据项

不是任何[实现][item]的成员的[数据项][implementation]，如*自由函数*或*自由常量*。常与[关联数据项][associated item]做对比。
<!-- An [item] that is not a member of an [implementation], such as a *free function* or a *free const*. Contrast to an [associated item]. -->

### Fundamental traits
基础trait

基础trait 就是如果为现有的类型再添加一个实现块（impl）就会带来破坏性的改变的 trait。`Fn` 和 `Sized` 这类 trait 就是基础trait。
<!-- A fundamental trait is one where adding an impl of it for an existing type is a breaking change. The `Fn` traits and `Sized` are fundamental. -->

### Fundamental type constructors
基本类型构造器

基本类型构造器是这样一种类型，在它之上实现一个 [blanket implementation](#blanket-implementation)是一个突破性的改变。`&`、`&mut`、`Box`、和 `Pin` 是基本类型构造器。
<!-- A fundamental type constructor is a type where implementing a [blanket implementation](#blanket-implementation) over it is a breaking change. `&`, `&mut`, `Box`, and `Pin`  are fundamental.  -->
如果任何时候 `T` 都被认为是[本地类型](#local-type)，那 `&T`、`&mut T`、`Box<T>`、和 `Pin<T>` 也被认为是本地类型。基本类型构造器不能[覆盖](#uncovered-type)其他类型。任何时候使用术语“有覆盖类型”时，都默认把`&T`、`&mut T`、`Box<T>`、和`Pin<T>` 排除在外。
<!-- Any time a type `T` is considered [local](#local-type), `&T`, `&mut T`, `Box<T>`, and `Pin<T>` are also considered local. Fundamental type constructors cannot [cover](#uncovered-type) other types. Any time the term "covered type" is used, the `T` in `&T`, `&mut T`, `Box<T>`, and `Pin<T>` is not considered covered. -->

### Inhabited
Inhabited

如果类型具有构造函数，因此可以实例化，则该类型是 inhabited。inhabited类型不是“空的”，因为可以有类型对应的值。相对的是 [Uninhabited](#uninhabited)。
<!-- A type is inhabited if it has constructors and therefore can be instantiated. An inhabited type is not "empty" in the sense that there can be values of the type. Opposite of [Uninhabited](#uninhabited). -->

### Inherent implementation

An [implementation] that applies to a nominal type, not to a trait-type pair.
[More][inherent implementation].

### Inherent method

A [method] defined in an [inherent implementation], not in a trait
implementation.

### Initialized

A variable is initialized if it has been assigned a value and hasn't since been
moved from. All other memory locations are assumed to be uninitialized. Only
unsafe Rust can create such a memory without initializing it.

### Local trait

A `trait` which was defined in the current crate. A trait definition is local
or not independent of applied type arguments. Given `trait Foo<T, U>`,
`Foo` is always local, regardless of the types substituted for `T` and `U`.

### Local type
本地类型

指在当前 crate 中定义的 `struct`、`enum`、或 `union` 。本地类型不会受到类型参数的影响。`struct Foo` 被认为是本地的，但 `Vec<Foo>` 不是。`LocalType<ForeignType>` 是本地的。类型别名不影响本地性。
<!-- A `struct`, `enum`, or `union` which was defined in the current crate. This is not affected by applied type arguments. `struct Foo` is considered local, but `Vec<Foo>` is not. `LocalType<ForeignType>` is local. Type aliases do not affect locality. -->

### Nominal types
标称类型

可用路径直接引用的类型。具体来说就是[枚举][enums]、[结构体][structs]、[联合体][unions]和 [trait对象][trait objects]。
<!-- Types that can be referred to by a path directly. Specifically [enums], [structs], [unions], and [trait objects]. -->

### Object safe traits
对象安全trait

可以用作 [trait 对象]的 [trait][Traits]。只有遵循特定[规则][object safety]的 trait 才是对象安全的。
<!-- [Traits] that can be used as [trait objects]. Only traits that follow specific [rules][object safety] are object safe. -->

### Prelude
预加载模块集

预加载模块集，或者 Rust 预加载模块集，是一个会被导入到每个 crate 中的每个模块的小型数据项集合（其中大部分是 trait）。trait 在预加载模块集中很普遍。
<!-- Prelude, or The Rust Prelude, is a small collection of items - mostly traits - that are imported into every module of every crate. The traits in the prelude are pervasive. -->

### Scrutinee
检验对象

检验对象是在`match`表达式和类似的模式匹配结构上匹配的表达式。例如，在 `match x { A => 1, B => 2 }` 中，表达式 `x` 是scrutinee。
<!-- A scrutinee is the expression that is matched on in `match` expressions and similar pattern matching constructs. For example, in `match x { A => 1, B => 2 }`, the expression `x` is the scrutinee. -->

### Size

The size of a value has two definitions.

The first is that it is how much memory must be allocated to store that value.

The second is that it is the offset in bytes between successive elements in an
array with that item type.

It is a multiple of the alignment, including zero. The size can change
depending on compiler version (as new optimizations are made) and target
platform (similar to how `usize` varies per-platform).

[More][alignment].

### Slice

A slice is dynamically-sized view into a contiguous sequence, written as `[T]`.

It is often seen in its borrowed forms, either mutable or shared. The shared
slice type is `&[T]`, while the mutable slice type is `&mut [T]`, where `T` represents
the element type.

### Statement
语句

语句是编程语言中最小的独立元素，它命令计算机执行一个动作。
<!-- A statement is the smallest standalone element of a programming language that commands a computer to perform an action. -->

### String literal
字符串字面量

字符串字面量是直接存储在最终二进制文件中的字符串，因此在 `'static` 有效期内是有效的。
<!-- A string literal is a string stored directly in the final binary, and so will be valid for the `'static` duration. -->

它的类型是 `'static`有效期的字符串切片借用，即：`&'static str`。
<!-- Its type is `'static` duration borrowed string slice, `&'static str`. -->

### String slice

A string slice is the most primitive string type in Rust, written as `str`. It is
often seen in its borrowed forms, either mutable or shared. The shared
string slice type is `&str`, while the mutable string slice type is `&mut str`.

Strings slices are always valid UTF-8.

### Trait

A trait is a language item that is used for describing the functionalities a type must provide.
It allows a type to make certain promises about its behavior.

Generic functions and generic structs can use traits to constrain, or bound, the types they accept.

### Uncovered type
无覆盖类型

不作为其他类型的参数出现的类型。例如，`T` 就是无覆盖的，但 `Vec<T>` 中的 `T` 就是有覆盖的。这（种说法）只与类型参数相关。
<!-- A type which does not appear as an argument to another type. For example, `T` is uncovered, but the `T` in `Vec<T>` is covered. This is only relevant for type arguments. -->

### Undefined behavior

Compile-time or run-time behavior that is not specified. This may result in,
but is not limited to: process termination or corruption; improper, incorrect,
or unintended computation; or platform-specific results.
[More][undefined-behavior].

### Uninhabited

如果类型没有构造函数，因此永远不能实例化，则该类型是无人居住的。一个无人居住的类型是“空的”，意思是该类型没有值。无居民类型的典型例子是[never type] ' !，或不带变体“enum Never{}”的枚举。相反(居住)(#居住
A type is uninhabited if it has no constructors and therefore can never be instantiated. An uninhabited type is "empty" in the sense that there are no values of the type. The canonical example of an uninhabited type is the [never type] `!`, or an enum with no variants `enum Never { }`. Opposite of [Inhabited](#inhabited).

[alignment]: type-layout.md#size-and-alignment
[associated item]: #associated-item
[enums]: items/enumerations.md
[free item]: #free-item
[implementation]: items/implementations.md
[implementations]: items/implementations.md
[inherent implementation]: items/implementations.md#固有实现
[item]: items.md
[method]: items/associated-items.md#methods
[never type]: types/never.md
[object safety]: items/traits.md#object-safety
[structs]: items/structs.md
[trait objects]: types/trait-object.md
[traits]: items/traits.md
[undefined-behavior]: behavior-considered-undefined.md
[unions]: items/unions.md
