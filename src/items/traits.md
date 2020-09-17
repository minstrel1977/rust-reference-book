# traite

>[traits.md](https://github.com/rust-lang/reference/blob/master/src/items/traits.md)\
>commit d5cc65a70f66a243d84cd251188d80fbe9926747

> **<sup>句法</sup>**\
> _Trait_ :\
> &nbsp;&nbsp; `unsafe`<sup>?</sup> `trait` [IDENTIFIER]&nbsp;
>              [_Generics_]<sup>?</sup>
>              ( `:` [_TypeParamBounds_]<sup>?</sup> )<sup>?</sup>
>              [_WhereClause_]<sup>?</sup> `{`\
> &nbsp;&nbsp;&nbsp;&nbsp; [_InnerAttribute_]<sup>\*</sup>\
> &nbsp;&nbsp;&nbsp;&nbsp; _TraitItem_<sup>\*</sup>\
> &nbsp;&nbsp; `}`
>
> _TraitItem_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> [_Visibility_]<sup>?</sup> (\
> &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; _TraitFunc_\
> &nbsp;&nbsp; &nbsp;&nbsp; | _TraitMethod_\
> &nbsp;&nbsp; &nbsp;&nbsp; | _TraitConst_\
> &nbsp;&nbsp; &nbsp;&nbsp; | _TraitType_\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_MacroInvocationSemi_]\
> &nbsp;&nbsp; )
>
> _TraitFunc_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _TraitFunctionDecl_ ( `;` | [_BlockExpression_] )
>
> _TraitMethod_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _TraitMethodDecl_ ( `;` | [_BlockExpression_] )
>
> _TraitFunctionDecl_ :\
> &nbsp;&nbsp; [_FunctionQualifiers_] `fn` [IDENTIFIER]&nbsp;[_Generics_]<sup>?</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp; `(` _TraitFunctionParameters_<sup>?</sup> `)`\
> &nbsp;&nbsp; &nbsp;&nbsp; [_FunctionReturnType_]<sup>?</sup> [_WhereClause_]<sup>?</sup>
>
> _TraitMethodDecl_ :\
> &nbsp;&nbsp; [_FunctionQualifiers_] `fn` [IDENTIFIER]&nbsp;[_Generics_]<sup>?</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp; `(` [_SelfParam_] (`,` _TraitFunctionParam_)<sup>\*</sup> `,`<sup>?</sup> `)`\
> &nbsp;&nbsp; &nbsp;&nbsp; [_FunctionReturnType_]<sup>?</sup> [_WhereClause_]<sup>?</sup>
>
> _TraitFunctionParameters_ :\
> &nbsp;&nbsp; _TraitFunctionParam_ (`,` _TraitFunctionParam_)<sup>\*</sup> `,`<sup>?</sup>
>
> _TraitFunctionParam_<sup>[†](#parameter-patterns)</sup> :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> ( [_Pattern_] `:` )<sup>?</sup> [_Type_]
>
> _TraitConst_ :\
> &nbsp;&nbsp; `const` [IDENTIFIER] `:` [_Type_]&nbsp;( `=` [_Expression_] )<sup>?</sup> `;`
>
> _TraitType_ :\
> &nbsp;&nbsp; `type` [IDENTIFIER] ( `:` [_TypeParamBounds_]<sup>?</sup> )<sup>?</sup> `;`

_trait_ 描述类型可以实现的抽象接口。该接口由[关联数据项]组成，关联数据项分三种类型:

- [函数](associated-items.md#associated-functions-and-methods)
- [类型](associated-items.md#associated-types)
- [常量](associated-items.md#associated-constants)

所有 trait 都定义了一个隐式类型参数 `Self` ，它指向“实现此接口的类型”。trait 还可能包含额外的类型参数。这些类型参数，包括 `Self` 在内，还可能会[像往常一样][泛型]受到其他特征的约束。

trait 是通过特定类型自己的[实现(implementations)]（代码段）来被这些类型实现的。

与 trait 相关的数据项不需要在该 trait 中提供具体定义，但也可以提供。如果 trait 提供了定义，那么该定义将作为任何不覆盖它的实现的默认值。如果没有提供，那么任何实现都必须提供具体定义。

## trait 约束

泛型约束的数据项可以使用 trait 作为其类型参数的[约束]。

## 泛型 trait

可以为 trait 指定类型参数，使其成为泛型。它们出现在 trait 名称之后，使用与[泛型函数]相同的语法。

```rust
trait Seq<T> {
    fn len(&self) -> u32;
    fn elt_at(&self, n: u32) -> T;
    fn iter<F>(&self, f: F) where F: Fn(T);
}
```

## 对象安全条款

对象安全的 trait 可以是 [trait对象] 的基础 trait。如果 trait 具有以下特性（在 [RFC 255] 中定义），则它是*对象安全的*：

* 它必须不能是 `Self: Sized`
* 所有的关联函数要么有 `where Self: Sized` 约束，要么
  * 不能有类型参数（生命周期参数不包括在内），并且
  * 作为[方法]不能使用直接 `Self`，除非 `Self` 是方法接收者内部的类型。
* 它必须没有任何关联常量。
* 所有的超类trait 也必须也是安全的。

当方法上没有 `Self: Sized` 绑定时，方法接收者的类型必须是以下类型之一：

* `&Self`
* `&mut Self`
* [`Box<Self>`]
* [`Rc<Self>`]
* [`Arc<Self>`]
* [`Pin<P>`] 当 `P` 是上面类型中的一种

```rust
# use std::rc::Rc;
# use std::sync::Arc;
# use std::pin::Pin;
// 对象安全的 trait 方法。
trait TraitMethods {
    fn by_ref(self: &Self) {}
    fn by_ref_mut(self: &mut Self) {}
    fn by_box(self: Box<Self>) {}
    fn by_rc(self: Rc<Self>) {}
    fn by_arc(self: Arc<Self>) {}
    fn by_pin(self: Pin<&Self>) {}
    fn with_lifetime<'a>(self: &'a Self) {}
    fn nested_pin(self: Pin<Arc<Self>>) {}
}
# struct S;
# impl TraitMethods for S {}
# let t: Box<dyn TraitMethods> = Box::new(S);
```

```rust,compile_fail
// 此 trait 是对象安全的，但不能在 trait对象上调度使用这些方法。
trait NonDispatchable {
    // 非方法不能被调度。
    fn foo() where Self: Sized {}
    // 在运行之前 Self 类型未知。
    fn returns(&self) -> Self where Self: Sized;
    // `other` 可能是另一具体类型的接收者。
    fn param(&self, other: Self) where Self: Sized {}
    // 泛型与虚函数指针表(Virtual Function Pointer Table, vtable)不兼容。
    fn typed<T>(&self, x: T) where Self: Sized {}
}

struct S;
impl NonDispatchable for S {
    fn returns(&self) -> Self where Self: Sized { S }
}
let obj: Box<dyn NonDispatchable> = Box::new(S);
obj.returns(); // 错误: 不能调用带有 Self 返回类型的方法
obj.param(S);  // 错误: 不能调用带有 Self 类型的参数的方法
obj.typed(1);  // 错误: 不能调用带有泛型类型的参数的方法
```

```rust,compile_fail
# use std::rc::Rc;
// 非对象安全的 trait
trait NotObjectSafe {
    const CONST: i32 = 1;  // 错误: 不能有关联常量

    fn foo() {}  // 错误: 关联函数没有 `Sized` 约束
    fn returns(&self) -> Self; // 错误: `Self` 在返回类型中 
    fn typed<T>(&self, x: T) {} // 错误: 泛型类型的参数
    fn nested(self: Rc<Box<Self>>) {} // 错误: 嵌套接受者还未被支持
}

struct S;
impl NotObjectSafe for S {
    fn returns(&self) -> Self { S }
}
let obj: Box<dyn NotObjectSafe> = Box::new(S); // 错误
```

```rust,compile_fail
// Self: Sized trait 非对象安全的。
trait TraitWithSize where Self: Sized {}

struct S;
impl TraitWithSize for S {}
let obj: Box<dyn TraitWithSize> = Box::new(S); // 错误
```

```rust,compile_fail
// 如果有 `Self` 这样的泛型参数，那 trait 就是非对象安全的
trait Super<A> {}
trait WithSelf: Super<Self> where Self: Sized {}

struct S;
impl<A> Super<A> for S {}
impl WithSelf for S {}
let obj: Box<dyn WithSelf> = Box::new(S); // 错误: 不能使用 `Self` 作为类型参数
```

## 超类trait

**超类trait** 是类型为了实现特定 trait 而需要（提前）实现的 trait。此外，任何地方一个[泛型]或 [trait对象]被某个 trait 约束，那这个泛型或 trait对象就可以访问这个*超类trait* 的关联数据项。

超类trait 是通过 trait 的 `Self` 类型上的 trait 约束来声明的，并且通过这种 trait 约束的方式来传递这种超类trait 声明关系。一个 trait 不能是它自己的超类trait。

有超类trait 的 trait 称其为其超类trait 的**子trait**。

下面是一个声明 `Shape` 是 `Circle` 的超类trait 的例子。

```rust
trait Shape { fn area(&self) -> f64; }
trait Circle : Shape { fn radius(&self) -> f64; }
```

下面是同一个示例，除了使用了 [where子句]。

```rust
trait Shape { fn area(&self) -> f64; }
trait Circle where Self: Shape { fn radius(&self) -> f64; }
```

下面例子通过 `Shape` 的 `area` 函数为 `radius` 提供了一个默认实现：

```rust
# trait Shape { fn area(&self) -> f64; }
trait Circle where Self: Shape {
    fn radius(&self) -> f64 {
        // 因为 A = pi * r^2
        // 所以通过代数推导
        // r = sqrt(A / pi)
        (self.area() /std::f64::consts::PI).sqrt()
    }
}
```

下一个示例调用了一个泛型参数的超类trait 的方法。
This next example calls a supertrait method on a generic parameter.
```rust
# trait Shape { fn area(&self) -> f64; }
# trait Circle : Shape { fn radius(&self) -> f64; }
fn print_area_and_radius<C: Circle>(c: C) {
    // 这里我们调用 `Circle` 的超类trait 的 area 方法。
    println!("Area: {}", c.area());
    println!("Radius: {}", c.radius());
}
```

类似地，这里是一个在 trait对象上调用 超类trait 的方法的例子。

```rust
# trait Shape { fn area(&self) -> f64; }
# trait Circle : Shape { fn radius(&self) -> f64; }
# struct UnitCircle;
# impl Shape for UnitCircle { fn area(&self) -> f64 { std::f64::consts::PI } }
# impl Circle for UnitCircle { fn radius(&self) -> f64 { 1.0 } }
# let circle = UnitCircle;
let circle = Box::new(circle) as Box<dyn Circle>;
let nonsense = circle.radius() * circle.area();
```

## 非安全trait

以关键字 `unsafe` 开头的 trait项表示*实现*该 trait 可能是[非安全]的。使用正确实现的非安全trait 是安全的。[trait实现]扔必须以关键字 `unsafe` 开头。

[`Sync`] 和 [`Send`] 是非安全trait。

## 参数模式

没有代码体的函数或方法声明只允许使用 [IDENTIFIER] 或 `_` [通配符][WildcardPattern]模式。当前 `mut` [IDENTIFIER]是允许的，但它已被弃用，将来将成为一个硬错误。
<!-- https://github.com/rust-lang/rust/issues/35203 -->

在2015版中，trait 的*函数或方法*的参数模式是可选的：

```rust
trait T {
    fn f(i32);  // 不需要参数的标识符.
}
```

所有的参数模式被限制为下述之一：

* [IDENTIFIER]
* `mut` [IDENTIFIER]
* [`_`][WildcardPattern]
* `&` [IDENTIFIER]
* `&&` [IDENTIFIER]

从2018版开始，*函数或方法*的参数模式不再是可选的。同时，只要有代码体，所有不可辩驳的模式都是被允许的。如果没有代码体，上面列出的限制仍然有效。

```rust,edition2018
trait T {
    fn f1((a, b): (i32, i32)) {}
    fn f2(_: (i32, i32));  // 没有代码体不能使用元组模式。
}
```

## 数据项可见性

trait项在语法上允许使用[_可见性_][_Visibility_]注释，但是当 trait 被验证时，这将被拒绝。这使得数据项可以在使用它们的不同上下文中使用统一的语法进行分析。例如，空的 `vis` 宏片段分类符可用于 trait项，其中宏规则可用于其他允许可见性的情况。

```rust
macro_rules! create_method {
    ($vis:vis $name:ident) => {
        $vis fn $name(&self) {}
    };
}

trait T1 {
    // 只允许空 `vis`。
    create_method! { method_of_t1 }
}

struct S;

impl S {
    // 这里允许使用可见性。
    create_method! { pub method_of_s }
}

impl T1 for S {}

fn main() {
    let s = S;
    s.method_of_t1();
    s.method_of_s();
}
```

[IDENTIFIER]: ../identifiers.md
[WildcardPattern]: ../patterns.md#wildcard-pattern
[_BlockExpression_]: ../expressions/block-expr.md
[_Expression_]: ../expressions.md
[_FunctionQualifiers_]: functions.md
[_FunctionReturnType_]: functions.md
[_Generics_]: generics.md
[_MacroInvocationSemi_]: ../macros.md#macro-invocation
[_OuterAttribute_]: ../attributes.md
[_InnerAttribute_]: ../attributes.md
[_Pattern_]: ../patterns.md
[_SelfParam_]: associated-items.md#methods
[_TypeParamBounds_]: ../trait-bounds.md
[_Type_]: ../types.md#type-expressions
[_Visibility_]: ../visibility-and-privacy.md
[_WhereClause_]: generics.md#where-clauses
[约束]: ../trait-bounds.md
[trait对象]: ../types/trait-object.md
[RFC 255]: https://github.com/rust-lang/rfcs/blob/master/text/0255-object-safety.md
[关联数据项]: associated-items.md
[方法]: associated-items.md#方法
[实现(implementations)]: implementations.md
[泛型]: generics.md
[where子句]: generics.md#where-clauses
[泛型函数]: functions.md#generic-functions
[非安全]: ../unsafety.md
[trait实现]: implementations.md#trait-implementations
[`Send`]: ../special-types-and-traits.md#send
[`Sync`]: ../special-types-and-traits.md#sync
[`Arc<Self>`]: ../special-types-and-traits.md#arct
[`Box<Self>`]: ../special-types-and-traits.md#boxt
[`Pin<P>`]: ../special-types-and-traits.md#pinp
[`Rc<Self>`]: ../special-types-and-traits.md#rct
