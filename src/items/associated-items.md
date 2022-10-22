# Associated Items
# 关联程序项/关联项

>[associated-items.md](https://github.com/rust-lang/reference/blob/master/src/items/associated-items.md)\
>commit: 6b9e4ffd539af7db5e27b6c829f120f827ef69a4 \
>本章译文最后维护日期：2022-10-22

> **<sup>句法</sup>**\
> _AssociatedItem_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> (\
> &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; [_MacroInvocationSemi_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | ( [_Visibility_]<sup>?</sup> ( [_TypeAlias_] | [_ConstantItem_] | [_Function_] ) )\
> &nbsp;&nbsp; )


*关联程序项*是在 [traits] 中声明或在[实现][implementations]中定义的程序项。之所以这样称呼它们，是因为它们是被定义在一个相关联的类型（即实现里指定的类型）上的。关联程序项是那些可在模块中声明的程序项的子集。具体来说，有[关联函数][associated functions]（包括方法）、[关联类型][associated types]和[关联常量][associated constants]。

[关联函数]: #associated-functions-and-methods
[关联类型]: #associated-types
[关联常量]: #associated-constants

当关联程序项与被关联程序项在逻辑上相关时，关联程序项就非常有用。例如，`Option` 上的 `is_some` 方法内在逻辑定义上就与 Option枚举类型相关，所以它应该和 Option 关联在一起。

每个关联程序项都有两种形式：（包含实际实现的）定义和（为定义声明签名的）声明。[^译者备注1]

正是这些声明构成了 trait 的契约(contract)以及其泛型参数中的可用内容。

## Associated functions and methods
## 关联函数和方法

*关联函数*是与一个类型相关联的[函数][functions]。

*关联函数声明*为*关联函数定义*声明签名。它的书写格式和函数项一样，除了函数体被替换为 `;`。

标识符是关联函数的名称。关联函数的泛型参数、参数列表、返回类型 和 where子句必须与它们在*关联函数声明*中声明的格式一致。

*关联函数定义*定义与另一个类型相关联的函数。它的编写方式与[函数项][function item]相同。

常见的关联函数的一个例子是 `new`函数，它返回此关联函数所关联的类型的值。

```rust
struct Struct {
    field: i32
}

impl Struct {
    fn new() -> Struct {
        Struct {
            field: 0i32
        }
    }
}

fn main () {
    let _struct = Struct::new();
}
```

当关联函数在 trait 上声明时，此函数也可以通过一个指向 trait，再后跟函数名的[路径][path]来调用。当发生这种情况时，可以用 trait 的实际路径和关联函数的标识符按 `<_ as Trait>::function_name` 这样的形式来组织实际的调用路径。

```rust
trait Num {
    fn from_i32(n: i32) -> Self;
}

impl Num for f64 {
    fn from_i32(n: i32) -> f64 { n as f64 }
}

// 在这个案例中，这4种形式都是等价的。
let _: f64 = Num::from_i32(42);
let _: f64 = <_ as Num>::from_i32(42);
let _: f64 = <f64 as Num>::from_i32(42);
let _: f64 = f64::from_i32(42);
```

### Methods
### 方法

如果关联函数的参数列表中的第一个参数名为 `self`[^译者备注2]，则此关联函数被称为*方法*，方法可以使用[方法调用操作符][method call operator](`.`)来调用，例如 `x.foo()`，也可以使用常用的函数调用形式进行调用。

如果名为 `self` 的参数的类型被指定了，它就通过以下文法（其中 `'lt` 表示生存期参数）来把此指定的参数限制解析成此文法中的一个类型：

```text
P = &'lt S | &'lt mut S | Box<S> | Rc<S> | Arc<S> | Pin<P>
S = Self | P
```

此文法中的 `Self`终结符(terminal)表示解析为实现类型(implementing type)的类型。这种解析包括解析上下文中的类型别名 `Self`、其他类型别名、或使用投射解析把（self 的类型中的）关联类型解析为实现类型。[^译者备注3]

> 译者注：原谅译者对上面这句背后知识的模糊理解，那首先给出原文：\
> The `Self` terminal in this grammar denotes a type resolving to the implementing type. This can also include the contextual type alias `Self`, other type aliases, or associated type projections resolving to the implementing type. \
> 译者在此先邀请读者中的高手帮忙翻译清楚。感谢感谢。另外译者还是要啰嗦以下译者对这句话背后知识的理解，希望有人能指出其中的错误，以让译者有机会进步：\
> 首先终结符(terminal)就是不能再更细分的词法单元，可以理解它是一个 token，这里它代表 self（即方法接受者）的类型的基础类型。上面句法中的 P 代表一个产生式，它内部定义的规则是并联的，就是自动机在应用这个产生式时碰到任意符合条件的输入就直接进入终态。S 代表有限自动机从 S 这里开始读取 self 的类型。这里 S 是 Self 和 P 的并联，应该表示是：如果 self 的类型直接是 Self，那就直接进入终态，即返回 Self，即方法接收者的直接类型就是结果类型；如果 self 的类型是 P 中的任一种，就返回那一种，比如 self 的类型是一个 box指针，那么就返回 `Box<S>`。


```rust
# use std::rc::Rc;
# use std::sync::Arc;
# use std::pin::Pin;
// 结构体 `Example` 上的方法示例
struct Example;
type Alias = Example;
trait Trait { type Output; }
impl Trait for Example { type Output = Example; }
impl Example {
    fn by_value(self: Self) {}
    fn by_ref(self: &Self) {}
    fn by_ref_mut(self: &mut Self) {}
    fn by_box(self: Box<Self>) {}
    fn by_rc(self: Rc<Self>) {}
    fn by_arc(self: Arc<Self>) {}
    fn by_pin(self: Pin<&Self>) {}
    fn explicit_type(self: Arc<Example>) {}
    fn with_lifetime<'a>(self: &'a Self) {}
    fn nested<'a>(self: &mut &'a Arc<Rc<Box<Alias>>>) {}
    fn via_projection(self: <Example as Trait>::Output) {}
}
```

（方法的首参）可以在不指定类型的情况下使用简写句法，具体对比如下：

简写模式               | 等效项
----------------------|-----------
`self`                | `self: Self`
`&'lifetime self`     | `self: &'lifetime Self`
`&'lifetime mut self` | `self: &'lifetime mut Self`

> **注意**: （方法的）生存期也能，其实也经常是使用这种方式来省略。

如果 `self` 参数以 `mut` 为前缀，它就变成了一个可变的变量，类似于使用 `mut` [标识符模式][identifier pattern]的常规参数。例如：

```rust
trait Changer: Sized {
    fn change(mut self) {}
    fn modify(mut self: Box<Self>) {}
}
```

以下是一个关于 trait 的方法的例子，现给定如下内容：

```rust
# type Surface = i32;
# type BoundingBox = i32;
trait Shape {
    fn draw(&self, surface: Surface);
    fn bounding_box(&self) -> BoundingBox;
}
```

这里定义了一个带有两个方法的 trait。当此 trait 被引入当前作用域内后，所有此 trait 的[实现][implementations]的值都可以调用此 trait 的 `draw` 和 `bounding_box` 方法。

```rust
# type Surface = i32;
# type BoundingBox = i32;
# trait Shape {
#     fn draw(&self, surface: Surface);
#     fn bounding_box(&self) -> BoundingBox;
# }
#
struct Circle {
    // ...
}

impl Shape for Circle {
    // ...
#   fn draw(&self, _: Surface) {}
#   fn bounding_box(&self) -> BoundingBox { 0i32 }
}

# impl Circle {
#     fn new() -> Circle { Circle{} }
# }
#
let circle_shape = Circle::new();
let bounding_box = circle_shape.bounding_box();
```

> **版次差异**: 在 2015 版中, 使用匿名参数来声明 trait方法是可能的 (例如：`fn foo(u8)`)。在 2018 版中，这已被弃用，再用会导致编译错误。新版次中所有的参数都必须有参数名。

#### Attributes on method parameters
#### 方法参数上的属性

方法参数上的属性遵循与[常规函数参数][regular function parameters]上相同的规则和限制。

## Associated Types
## 关联类型

*关联类型*是与另一个类型关联的[类型别名(type aliases)][type aliases] 。关联类型不能在[固有实现][inherent implementations]中定义，也不能在 trait 中给它们一个默认实现。

*关联类型声明*为关联类型声明其签名。
书写形式为下面这几种类型，其中 `Assoc` 是关联类型的名称，`Params` 是逗号分割的由类型、生存期或常量组成的参数列表，`Bounds` 是由加号分割的且此关联类型必须受约的 trait约束表，`WhereBounds` 是逗号分割的且此关联类型必须受约的 trait约束表。

<!-- ignore: illustrative example forms -->
```rust,ignore
type Assoc;
type Assoc: Bounds;
type Assoc<Params>;
type Assoc<Params>: Bounds;
type Assoc<Params> where WhereBounds;
type Assoc<Params>: Bounds where WhereBounds;
```

关联类型里的标识符是其声明的类型的别名的名称；其可选的 trait约束必须由此类型别名的实现(implementations)来履行实现。
关联类型上有一个隐式的 [`Sized`]约束，可以使用 `？Sized`放宽此约束。

*关联类型定义*定义了用于在类型上实现特定trait 的类型别名。它们的书写形式类似于*关联类型声明*，但不能包含类似于上面的 `Bounds`，并且必须包含一个类似于下面的 `Type`：

<!-- ignore: illustrative example forms -->
```rust,ignore
type Assoc = Type;
type Assoc<Params> = Type; // 这里的 `Type` 的类型可以引用 `Params` 作为类型参数
type Assoc<Params> = Type where WhereBounds;
type Assoc<Params> where WhereBounds = Type; // 已经不推荐这种写法

如果类型 `Item` 上有一个来自 trait `Trait`的关联类型 `Assoc`，则表达式 `<Item as Trait>::Assoc` 也是一个类型，具体就是*关联类型定义*中指定的类型的一个别名。此外，如果 `Item` 是类型参数，则 `Item::Assoc` 也可以在类型参数中使用。

关联类型可包括[泛型参数][generic parameters]和 [where子句][where clauses]；这些通常被称为*泛型关联类型*或 *GATs*。如果类型`Thing` 带有一个从 带有泛型参数`<'a>` 的`Trait`中继承过来的关联类型`Item`，则该类型可以命名为 `<Thing as Trait>::Item<'x>`，其中 `'x` 是作用域`'a` 内的某个生存期。此时，在此关联类型定义的impls 中出现的 `'a` 将会被 `'x` 替换。

```rust
trait AssociatedType {
    // 关联类型声明
    type Assoc;
}

struct Struct;

struct OtherStruct;

impl AssociatedType for Struct {
    // 关联类型定义
    type Assoc = OtherStruct;
}

impl OtherStruct {
    fn new() -> OtherStruct {
        OtherStruct
    }
}

fn main() {
    // 使用 <Struct as AssociatedType>::Assoc 来引用关联类型 OtherStruct
    let _other_struct: OtherStruct = <Struct as AssociatedType>::Assoc::new();
}
```

下面是一个带有泛型和 where子句的关联类型的示例：

```rust
struct ArrayLender<'a, T>(&'a mut [T; 16]);

trait Lend {
    // 泛型关联类型生命
    type Lender<'a> where Self: 'a;
    fn lend<'a>(&'a mut self) -> Self::Lender<'a>;
}

impl<T> Lend for [T; 16] {
    // 泛型关联类型定义
    type Lender<'a> = ArrayLender<'a, T> where Self: 'a;

    fn lend<'a>(&'a mut self) -> Self::Lender<'a> {
        ArrayLender(self)
    }
}

fn borrow<'a, T: Lend>(array: &'a mut T) -> <T as Lend>::Lender<'a> {
    array.lend()
}


fn main() {
    let mut array = [0usize; 16];
    let lender = borrow(&mut array);
}
```

### Associated Types Container Example
### 示例展示容器类型的关联类型

下面给出一个 `Container` trait 示例。请注意，该类型可用在方法签名内：

```rust
trait Container {
    type E;
    fn empty() -> Self;
    fn insert(&mut self, elem: Self::E);
}
```

为了能让实现类型来实现此 trait，实现类型不仅必须为每个方法提供实现，而且必须指定类型 `E`。下面是一个为标准库类型 `Vec` 实现了此 `Container` 的实现：

```rust
# trait Container {
#     type E;
#     fn empty() -> Self;
#     fn insert(&mut self, elem: Self::E);
# }
impl<T> Container for Vec<T> {
    type E = T;
    fn empty() -> Vec<T> { Vec::new() }
    fn insert(&mut self, x: T) { self.push(x); }
}
```

### Relationship between `Bounds` and `WhereBounds`
### `Bounds` 和 `WhereBounds` 之间的关系

在下面这个例子里：

```rust
# use std::fmt::Debug;
trait Example {
    type Output<T>: Ord where T: Debug;
}
```

假定有一个对关联类型的引用，如 `<X as Example>::Output<Y>`，关联类型本身必须受 `Ord` 的约束，而类型`Y` 必须受 `Debug` 的约束。

### Required where clauses on generic associated types
### 泛型关联类型中的 where子句的前提条件

trait 上的的泛型关联类型声明目前可能需要一个 where子句列表，这取决于 trait 中的函数以及 GAT 如何被使用。这些规则将来可能会放宽；可以在[泛型关联类型初创提案仓库](https://rust-lang.github.io/generic-associated-types-initiative/explainer/required_bounds.html)中找到最新信息。

简言之，where子句是必需的，这种书写方式可以最有可能的在各类 impls 中定义关联类型。要做到这一点，当 GAT 作为函数的输入或输出时，此函数里提供的任何*合法*的子句在之前的 GAT 中必须也写一遍。

```rust
trait LendingIterator {
    type Item<'x> where Self: 'x;
    fn next<'a>(&'a mut self) -> Self::Item<'a>;
}
```

在上面的 `next`函数中，我们提供了一个 `Self: 'a` 子句，因为有 `&'a mut self` 限制；因此，我们必须在 GAT 本身上写一个等价的约束：`where Self: 'x`。

当 trait 中有多个函数使用同一个 GAT 时，则此 GAT 使用这些函数的约束的*交集*，而不是并集。（译者注：注意生存期的长短关系和父子关系是逆向的）

```rust
trait Check<T> {
    type Checker<'x>;
    fn create_checker<'a>(item: &'a T) -> Self::Checker<'a>;
    fn do_check(checker: Self::Checker<'_>);
}
```

在本例中，`type Checker<'a>;` 不需要任何约束。虽然我们知道 `create_checker` 上有 `T: 'a`，但我们不知道 `do_check` 上有什么约束。但是，如果 `do_check` 被注释掉，则 `Checker` 上需要做 `where T: 'x`约束。

关联类型上的约束还会传播其所需的where子句。

```rust
trait Iterable {
    type Item<'a> where Self: 'a;
    type Iterator<'a>: Iterator<Item = Self::Item<'a>> where Self: 'a;
    fn iter<'a>(&'a self) -> Self::Iterator<'a>;
}
```

这里， 因为函数`iter`，`Item` 需要 `where Self: 'a`。但是，在 `Iterator` 中，因为 `Item` 被用到了，那么 `Item` 上的 `where Self: 'a`约束也要在这里声明一遍。

最后，在 trait 中，明确使用 `'static` 的 GAT 不需要明确声明其约束条件。

```rust
trait StaticReturn {
    type Y<'a>;
    fn foo(&self) -> Self::Y<'static>;
}
```

## Associated Constants
## 关联常量

*关联常量*是与具体类型关联的[常量][constants]。

*关联常量声明*为*关联常量定义*声明签名。书写形式为：先是 `const` 开头，然后是标识符，然后是 `:`，然后是一个类型，最后是一个 `;`。

这里标识符是（外部引用）路径中使用的常量的名称；类型是（此关联常量的）定义必须实现的类型。

*关联常量定义*定义了与类型关联的常量。它的书写方式与[常量项][constant item]相同。

关联常量定义仅在引用时进行[常量求值][constant evaluation]。此外，包含[泛型参数][generic parameters]的关联常量定义在单态化后才进行求值。

```rust,compile_fail
struct Struct;
struct GenericStruct<const ID: i32>;

impl Struct {
    // 定义不会立即执行求值操作
    const PANIC: () = panic!("compile-time panic");
}

impl<const ID: i32> GenericStruct<ID> {
    // 定义不会立即执行求值操作
    const NON_ZERO: () = if ID == 0 {
        panic!("contradiction")
    };
}

fn main() {
    // 引用 Struct::PANIC 导致编译错误
    let _ = Struct::PANIC;

    // 可以编译通过, ID 非 0
    let _ = GenericStruct::<1>::NON_ZERO;

    // 编译错误，因为使用 ID=0 来求值计算 NON_ZERO  
    let _ = GenericStruct::<0>::NON_ZERO;
}
```

### Associated Constants Examples
### 示例展示关联常量

基本示例：

```rust
trait ConstantId {
    const ID: i32;
}

struct Struct;

impl ConstantId for Struct {
    const ID: i32 = 1;
}

fn main() {
    assert_eq!(1, Struct::ID);
}
```

使用默认值：

```rust
trait ConstantIdDefault {
    const ID: i32 = 1;
}

struct Struct;
struct OtherStruct;

impl ConstantIdDefault for Struct {}

impl ConstantIdDefault for OtherStruct {
    const ID: i32 = 5;
}

fn main() {
    assert_eq!(1, Struct::ID);
    assert_eq!(5, OtherStruct::ID);
}
```

[^译者备注1]: 固有实现中声明和定义是在一起的。

[^译者备注2]: 把简写形式转换成等价的标准形式。

[^译者备注3]: 结合下面的示例理解。

[_ConstantItem_]: constant-items.md
[_Function_]: functions.md
[_MacroInvocationSemi_]: ../macros.md#macro-invocation
[_OuterAttribute_]: ../attributes.md
[_TypeAlias_]: type-aliases.md
[_Visibility_]: ../visibility-and-privacy.md
[`Arc<Self>`]: ../special-types-and-traits.md#arct
[`Box<Self>`]: ../special-types-and-traits.md#boxt
[`Pin<P>`]: ../special-types-and-traits.md#pinp
[`Rc<Self>`]: ../special-types-and-traits.md#rct
[`Sized`]: ../special-types-and-traits.md#sized
[traits]: traits.md
[type aliases]: type-aliases.md
[inherent implementations]: implementations.md#inherent-implementations
[identifier]: ../identifiers.md
[identifier pattern]: ../patterns.md#identifier-patterns
[implementations]: implementations.md
[type]: ../types.md#type-expressions
[constants]: constant-items.md
[constant item]: constant-items.md
[functions]: functions.md
[function item]: ../types/function-item.md
[method call operator]: ../expressions/method-call-expr.md
[path]: ../paths.md
[regular function parameters]: functions.md#attributes-on-function-parameters
[generic parameters]: generics.md
[where clauses]: generics.md#where-clauses
[constant evaluation]: ../const_eval.md