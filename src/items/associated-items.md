# 关联数据项

>[associated-items.md](https://github.com/rust-lang/reference/blob/master/src/items/associated-items.md)\
>commit: 136bd7da8b9c509c17c9619813b57dd1a47a8e25

*关联数据项*是在[trait]中声明或在[实现]中定义的数据项。之所以这样称呼它们，是因为它们是在关联类型上定义的，即实现里的类型。它们是可以在模块中声明的数据项的子集。具体来说，有[关联函数]（包括方法）、[关联类型]和[关联常量]。

[关联函数]: #关联函数和类型
[关联类型]: #关联类型
[关联常量]: #关联常量

当关联数据项与被关联数据项在逻辑上相关时，关联数据项非常有用。例如，`Option` 上的 `is_some` 方法与 Options 相关，所以应该关联。（译者注：这句翻译过来实在是怪异，那先TobeModify吧 Associated items are useful when the associated item logically is related to the associating item. For example, the `is_some` method on `Option` is intrinsically related to Options, so should be associated.）

每一种关联数据项都有两种形式：包含实际实现的定义和声明定义签名的声明。

正是这些声明构成了 trait 的契约以及泛型类型中可用的内容。

## 关联函数和方法

*关联函数*是与类型相关联的[函数]。

*关联函数声明*为*关联函数定义*声明签名。它被写为函数项，但函数体被替换为 `;`。

标识符是函数的名称。关联函数的泛型、参数列表、返回类型和 where子句必须与关联函数声明中的这些相同。

*关联函数定义*定义与另一个类型关联的函数。它的编写方式与[函数项]相同。

常见的关联函数的一个例子是 `new` 函数，它返回与关联函数关联的类型的值。

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

当关联函数在一个 trait 上声明时，此函数也可以通过一个[路径]来调用，这个路径是一个附加了 trait 名字的 trait 的路径。当发生这种情况时，可以用实际的路径和标识符按 `<_ as Trait>::function_name` 这样的形式来组织实际的调用路径。

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

参数列表中的第一个参数名为 `self` 的关联函数被称为*方法*，可以使用[方法调用操作符]来调用（例如 `x.foo()`）以及常用的函数调用形式进行调用。

如果 `self` 参数的类型被指定，它就被限制解析为以下语法生成的类型(其中 `'lt` 表示某个任意的生存期)：

```text
P = &'lt S | &'lt mut S | Box<S> | Rc<S> | Arc<S> | Pin<P>
S = Self | P
```

此语法中的 `Self` 表示对实现类型的类型解析。这还可以包括上下文类型别名 `Self`、其他类型别名或对实现类型的关联类型预测解析。（译者注：这句对我来说太难了，只能先加TobeModify了。顺便奉上原文：The `Self` terminal in this grammar denotes a type resolving to the implementing type. This can also include the contextual type alias `Self`, other type aliases, or associated type projections resolving to the implementing type.）

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

（方法的首参）可以在不指定类型的情况下使用简写句法，具体如下：

简写模式               | 等效项
----------------------|-----------
`self`                | `self: Self`
`&'lifetime self`     | `self: &'lifetime Self`
`&'lifetime mut self` | `self: &'lifetime mut Self`

> **注意**: （方法的）生存期也能，其实也经常是使用这种方式来省略。

如果 `self` 参数以 `mut` 为前缀，它就变成了一个可变的变量，类似于使用 `mut` [标识符模式]的常规参数。例如：

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

这定义了一个带有两个方法的trait。当此 trait 在作用域内时，所有具有此 trait[实现]的值都可以调用它们的 `draw` 和 `bounding_box` 方法。

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

> **版本差异**: 在 2015 版种, 使用匿名参数声明 trait方法是可能的 (例如：`fn foo(u8)`)。在 2018 版本中，这被弃用，再用会导致编译错误。新版本种所有的参数都必须有参数名。

#### 方法参数上的属性

方法参数上的属性遵循与[常规函数参数]相同的规则和限制。

## 关联类型

*关联类型*是与另一个类型关联的[类型别名]。关联类型不能在[固有实现]中定义，也不能在 trait 中给它们一个默认实现。

*关联类型声明*为*关联类型定义*声明签名。书写形式为：先是 `type`，然后是一个[标识符]，最后是一个可选的 trait约束列表。

这里标识符是声明的类型别名的名称；可选的 trait约束必须由类型别名的实现来满足。

*关联类型定义*在另一个类型上定义一个类型别名。书写形式为：先是 `type`，然后是一个[标识符]，然后再是一个 `=`，最后是一个[类型]。

如果类型 `Item` 具有来自 trait `Trait`的关联类型 `Assoc`，则表达式 `<Item as Trait>::Assoc` 也是一个类型，具体就是关联类型定义中指定的类型的一个别名。此外，如果 `Item` 是类型参数，则 `Item::Assoc` 也可用作类型参数。

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

### 示例展示容器内的关联类型

下面给出一个 `Container` trait 示例。请注意，该类型可用于方法签名：

```rust
trait Container {
    type E;
    fn empty() -> Self;
    fn insert(&mut self, elem: Self::E);
}
```

为了使类型实现此 trait，它不仅必须为每个方法提供实现，而且必须指定类型 `E`。下面是标准库类型 `Vec` 的 `Container` 实现：

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

## 关联常量

*关联常量*是与类型关联的[常量]。

*关联常量声明*为关联常量定义声明签名。书写形式为：先是 `const` 开头，然后是标识符，然后是 `:`，然后是一个类型，最后是一个 `;`。

这里标识符是路径中使用的常量的名称；类型是（关联常量）定义必须实现的类型。

*关联常量定义*定义与类型关联的常量。它的编写方式与[常量项]相同。

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
[_WhereClause_]: generics.md#where子句
[`Arc<Self>`]: ../special-types-and-traits.md#arct
[`Box<Self>`]: ../special-types-and-traits.md#boxt
[`Pin<P>`]: ../special-types-and-traits.md#pinp
[`Rc<Self>`]: ../special-types-and-traits.md#rct
[_OuterAttribute_]: ../attributes.md
[trait]: traits.md
[类型别名]: type-aliases.md
[固有实现]: implementations.md#固有实现
[标识符]: ../identifiers.md
[标识符模式]: ../patterns.md#identifier-patterns
[实现]: implementations.md
[类型]: ../types.md#type-expressions
[常量]: constant-items.md
[常量项]: constant-items.md
[函数]: functions.md
[函数项]: ../types/function-item.md
[方法调用操作符]: ../expressions/method-call-expr.md
[路径]: ../paths.md
[常规函数参数]: functions.md#函数参数上的属性
