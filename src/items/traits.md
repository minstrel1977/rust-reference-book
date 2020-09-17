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

对象安全的 trait 可以是 [trait 对象] 的基础 trait。如果 trait 具有以下特性（在 [RFC 255] 中定义），则它是*对象安全的*：

* 它必须不能是 `Self: Sized`
* 所有的关联函数要么有 `where Self: Sized` 约束，要么
  * 不能有类型参数（生命周期参数不包括在内），并且
  * 作为[方法]不能使用直接 `Self`，除非 `Self` 是方法接收者内部的类型。
* 它必须没有任何关联常量。
* 所有的超类 trait 也必须也是安全的。

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
// 此 trait 是对象安全的，但不能在 trait 对象上调度使用这些方法。
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

## 超类 trait

**超类 trait** 是类型为了实现特定 trait 而需要提前实现的 trait。此外，如果[泛型]或 [*trait 对象*]被某个 trait 绑定，那这个 trait 就可以访问这些对象的*超类 trait* 的关联数据项。

Supertraits are declared by trait bounds on the `Self` type of a trait and
transitively the supertraits of the traits declared in those trait bounds. It is
an error for a trait to be its own supertrait.

The trait with a supertrait is called a **subtrait** of its supertrait.

The following is an example of declaring `Shape` to be a supertrait of `Circle`.

```rust
trait Shape { fn area(&self) -> f64; }
trait Circle : Shape { fn radius(&self) -> f64; }
```

And the following is the same example, except using [where clauses].

```rust
trait Shape { fn area(&self) -> f64; }
trait Circle where Self: Shape { fn radius(&self) -> f64; }
```

This next example gives `radius` a default implementation using the `area`
function from `Shape`.

```rust
# trait Shape { fn area(&self) -> f64; }
trait Circle where Self: Shape {
    fn radius(&self) -> f64 {
        // A = pi * r^2
        // so algebraically,
        // r = sqrt(A / pi)
        (self.area() /std::f64::consts::PI).sqrt()
    }
}
```

This next example calls a supertrait method on a generic parameter.

```rust
# trait Shape { fn area(&self) -> f64; }
# trait Circle : Shape { fn radius(&self) -> f64; }
fn print_area_and_radius<C: Circle>(c: C) {
    // Here we call the area method from the supertrait `Shape` of `Circle`.
    println!("Area: {}", c.area());
    println!("Radius: {}", c.radius());
}
```

Similarly, here is an example of calling supertrait methods on trait objects.

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

## Unsafe traits

Traits items that begin with the `unsafe` keyword indicate that *implementing* the
trait may be [unsafe]. It is safe to use a correctly implemented unsafe trait.
The [trait implementation] must also begin with the `unsafe` keyword.

[`Sync`] and [`Send`] are examples of unsafe traits.

## Parameter patterns

Function or method declarations without a body only allow [IDENTIFIER] or
`_` [wild card][WildcardPattern] patterns. `mut` [IDENTIFIER] is currently
allowed, but it is deprecated and will become a hard error in the future.
<!-- https://github.com/rust-lang/rust/issues/35203 -->

In the 2015 edition, the pattern for a trait function or method parameter is
optional:

```rust
trait T {
    fn f(i32);  // Parameter identifiers are not required.
}
```

The kinds of patterns for parameters is limited to one of the following:

* [IDENTIFIER]
* `mut` [IDENTIFIER]
* [`_`][WildcardPattern]
* `&` [IDENTIFIER]
* `&&` [IDENTIFIER]

Beginning in the 2018 edition, function or method parameter patterns are no
longer optional. Also, all irrefutable patterns are allowed as long as there
is a body. Without a body, the limitations listed above are still in effect.

```rust,edition2018
trait T {
    fn f1((a, b): (i32, i32)) {}
    fn f2(_: (i32, i32));  // Cannot use tuple pattern without a body.
}
```

## Item visibility

Trait items syntactically allow a [_Visibility_] annotation, but this is
rejected when the trait is validated. This allows items to be parsed with a
unified syntax across different contexts where they are used. As an example,
an empty `vis` macro fragment specifier can be used for trait items, where the
macro rule may be used in other situations where visibility is allowed.

```rust
macro_rules! create_method {
    ($vis:vis $name:ident) => {
        $vis fn $name(&self) {}
    };
}

trait T1 {
    // Empty `vis` is allowed.
    create_method! { method_of_t1 }
}

struct S;

impl S {
    // Visibility is allowed here.
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
[bounds]: ../trait-bounds.md
[trait object]: ../types/trait-object.md
[RFC 255]: https://github.com/rust-lang/rfcs/blob/master/text/0255-object-safety.md
[associated items]: associated-items.md
[method]: associated-items.md#methods
[实现(implementations)]: implementations.md
[generics]: generics.md
[where clauses]: generics.md#where-clauses
[generic functions]: functions.md#generic-functions
[unsafe]: ../unsafety.md
[trait implementation]: implementations.md#trait-implementations
[`Send`]: ../special-types-and-traits.md#send
[`Sync`]: ../special-types-and-traits.md#sync
[`Arc<Self>`]: ../special-types-and-traits.md#arct
[`Box<Self>`]: ../special-types-and-traits.md#boxt
[`Pin<P>`]: ../special-types-and-traits.md#pinp
[`Rc<Self>`]: ../special-types-and-traits.md#rct
