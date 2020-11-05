# Associated Items
# 关联数据项/关联项

>[associated-items.md](https://github.com/rust-lang/reference/blob/master/src/items/associated-items.md)\
>commit: 136bd7da8b9c509c17c9619813b57dd1a47a8e25 \
>本译文最后维护日期：2020-10-22

*关联数据项*是在 [traits] 中声明或在[实现][implementations]中定义的数据项。之所以这样称呼它们，是因为它们的定义是在类型的关联项里定义的，即类型的实现里定义的。它们是可在模块中声明的数据项的子集。具体来说，有[关联函数][associated functions]（包括方法）、[关联类型][associated types]和[关联常量][associated constants]。

[关联函数]: #associated-functions-and-methods
[关联类型]: #associated-types
[关联常量]: #associated-constants

当关联数据项与被关联数据项在逻辑上相关时，关联数据项非常有用。例如，`Option` 上的 `is_some` 方法与 Options 是内在相关的，所以它们应该关联在一起。

每一种关联数据项都有两种形式：（包含实际实现的）定义和（为定义声明签名的）声明。

正是这些声明构成了 trait 的约定(contract)以及其泛型参数中可用的资源。

## Associated functions and methods
## 关联函数和方法

*关联函数*是与具体类型相关联的[函数][functions]。

*关联函数声明*为*关联函数定义*声明签名。它的书写格式和函数项一样，除了函数体被替换为 `;`。

标识符是函数的名称。关联函数的泛型参数、参数列表、返回类型 和 where子句必须与它们在*关联函数声明*中声明的格式一致。

*关联函数定义*定义与类型相关联的函数。它的编写方式与[函数项][function item]相同。

常见的关联函数的一个例子是 `new` 函数，它返回此关联函数所关联的类型的值。

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

当关联函数在 trait 上声明时，此函数也可以通过指向 trait，再后跟函数名的[路径][path]来调用。当发生这种情况时，可以用实际的路径和标识符按 `<_ as Trait>::function_name` 这样的形式来组织实际的调用路径。

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

> _Method_ :\
> &nbsp;&nbsp; [_FunctionQualifiers_] `fn` [IDENTIFIER]&nbsp;[_Generics_]<sup>?</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp; `(` _SelfParam_ (`,` [_FunctionParam_])<sup>\*</sup> `,`<sup>?</sup> `)`\
> &nbsp;&nbsp; &nbsp;&nbsp; [_FunctionReturnType_]<sup>?</sup> [_WhereClause_]<sup>?</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp; [_BlockExpression_]
>
> _SelfParam_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> ( _ShorthandSelf_ | _TypedSelf_ )
>
> _ShorthandSelf_ :\
> &nbsp;&nbsp;  (`&` | `&` [_Lifetime_])<sup>?</sup> `mut`<sup>?</sup> `self`
>
> _TypedSelf_ :\
> &nbsp;&nbsp; `mut`<sup>?</sup> `self` `:` [_Type_]

参数列表中的第一个参数名为 `self` 的关联函数被称为*方法*，方法可以使用[方法调用操作符][method call operator]来调用，例如 `x.foo()`，也可以使用常用的函数调用形式进行调用。

如果 `self` 参数的类型被指定，它就通过以下语法(其中 `'lt` 表示生存期参数)被限制解析成几个相关类型中的一个：

```text
P = &'lt S | &'lt mut S | Box<S> | Rc<S> | Arc<S> | Pin<P>
S = Self | P
```

此语法中的 `Self` 代表对实现类型(implementing type)的类型解析。此解析还可以包括对上下文中的类型别名 `Self`、其他类型别名、或对实现类型采用关联类型预测解析方法解析。（原文：The `Self` terminal in this grammar denotes a type resolving to the implementing type. This can also include the contextual type alias `Self`, other type aliases, or associated type projections resolving to the implementing type. 译者注：这句对译者来说太难了，只能先这么将就着翻译，同时放出原文，请读者中的高手帮忙翻译清楚。感谢感谢）

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

这里定义了一个带有两个方法的 trait。当此 trait 被引入当前作用域内后，所有拥有此 trait的[实现][implementations]的值都可以调用此 trait 的 `draw` 和 `bounding_box` 方法。

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

> **版本差异**: 在 2015 版中, 使用匿名参数来声明 trait方法是可能的 (例如：`fn foo(u8)`)。在 2018 版本中，这已被弃用，再用会导致编译错误。新版本种所有的参数都必须有参数名。

#### Attributes on method parameters
#### 方法参数上的属性

方法参数上的属性遵循与[常规函数参数][regular function parameters]上相同的规则和限制。

## Associated Types
## 关联类型

*关联类型*是与另一个类型关联的[类型别名][type aliases] 。关联类型不能在[固有实现][inherent implementations]中定义，也不能在 trait 中给它们一个默认实现。

*关联类型声明*为*关联类型定义*声明签名。书写形式为：先是 `type`，然后是一个[标识符][identifier]，最后是一个可选的 trait约束列表。

这里的标识符是声明的类型的别名名称；可选的 trait约束必须由此类型别名的实现来履行。

*关联类型定义*在另一个类型上定义了一个类型别名。书写形式为：先是 `type`，然后是一个[标识符]，然后再是一个 `=`，最后是一个[类型][type]。

如果类型 `Item` 有一个来自 trait `Trait`的关联类型 `Assoc`，则表达式 `<Item as Trait>::Assoc` 也是一个类型，具体就是*关联类型定义*中指定的类型的一个别名。此外，如果 `Item` 是类型参数，则 `Item::Assoc` 也可以在类型参数中使用。

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

### Associated Types Container Example
### 示例展示容器内的关联类型

下面给出一个 `Container` trait 示例。请注意，该类型可用在方法签名内：

```rust
trait Container {
    type E;
    fn empty() -> Self;
    fn insert(&mut self, elem: Self::E);
}
```

为了使类型实现此 trait，它不仅必须为每个方法提供实现，而且必须指定类型 `E`。下面是一个为标准库类型 `Vec` 实现了 `Container` 的实现：

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

## Associated Constants
## 关联常量

*关联常量*是与具体类型关联的[常量][constants]。

*关联常量声明*为*关联常量定义*声明签名。书写形式为：先是 `const` 开头，然后是标识符，然后是 `:`，然后是一个类型，最后是一个 `;`。

这里标识符是（外部引用）路径中使用的常量的名称；类型是（关联常量）定义必须实现的类型。

*关联常量定义*定义了与类型关联的常量。它的编写方式与[常量项][constant item]相同。

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

[_BlockExpression_]: ../expressions/block-expr.md
[_FunctionParam_]: functions.md
[_FunctionQualifiers_]: functions.md
[_FunctionReturnType_]: functions.md
[_Generics_]: generics.md
[_Lifetime_]: ../trait-bounds.md
[_Type_]: ../types.md#type-expressions
[_WhereClause_]: generics.md#where-clauses
[`Arc<Self>`]: ../special-types-and-traits.md#arct
[`Box<Self>`]: ../special-types-and-traits.md#boxt
[`Pin<P>`]: ../special-types-and-traits.md#pinp
[`Rc<Self>`]: ../special-types-and-traits.md#rct
[_OuterAttribute_]: ../attributes.md
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

<!-- 2020-11-3 -->
<!-- checked -->
