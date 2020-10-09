# Special types and traits
# 特殊类型和 trait

>[special-types-and-traits.md](https://github.com/rust-lang/reference/blob/master/src/special-types-and-traits.md)\
>commit fcdc0cab546c10921d66054be25c6afc9dd6b3bc

存在于[标准库][the standard library]中的某些类型和 trait 是 Rust 编译器默认知道的。本章阐述了这些类型和 trait 的特性。

## `Box<T>`

[`Box<T>`] 有一些特殊的特性，Rust目前还不允许用户定义类型时使用。
has a few special features that Rust doesn't currently allow for user defined types.

* `Box<T>` 的[解引用操作符]会产生一个可移动的位置。这意味着 `*`运算符和 `Box<T>` 的析构函数是语言内置的。
* [方法][Methods]可以用 `Box<Self>` 作为接收者。
* `Box<T>`可以在与 `T` 在同一 crate 中实现同一 trait，[孤儿规则][orphan rules]禁止其他泛型类型这么做。

## `Rc<T>`

[方法][Methods]可以用 [`Rc<Self>`] 作为接收者。

## `Arc<T>`

[方法][Methods]可以用 [`Arc<Self>`] 作为接收者。

## `Pin<P>`

[方法][Methods]可以用 [`Pin<P>`] 作为接收者。

## `UnsafeCell<T>`

[`std::cell::UnsafeCell<T>`] 用于[内部可变性][interior mutability]。它确保编译器不会对此类类型执行不正确的优化。它还确保具有内部可变类型的[静态(`static`)项][`static` items]不会被放在标记为只读的内存中。

## `PhantomData<T>`

[`std::marker::PhantomData<T>`] 是一个零尺寸零最小对齐量，且被认为拥有 `T` 的类型，这个类型存在目的是应用在[型变][variance]，[销毁检查][drop check]，和[自动trait](#auto-traits)中的。

## Operator Traits
## 运算符trait

The traits in [`std::ops`] and [`std::cmp`] are used to overload [operators],
[indexing expressions], and [call expressions].

## `Deref` and `DerefMut`

As well as overloading the unary `*` operator, [`Deref`] and [`DerefMut`] are
also used in [method resolution] and [deref coercions].

## `Drop`

The [`Drop`] trait provides a [destructor], to be run whenever a value of this
type is to be destroyed.

## `Copy`

The [`Copy`] trait changes the semantics of a type implementing it. Values
whose type implements `Copy` are copied rather than moved upon assignment.

`Copy` can only be implemented for types which do not implement `Drop`, and whose fields are all `Copy`.
For enums, this means all fields of all variants have to be `Copy`.
For unions, this means all variants have to be `Copy`.

`Copy` is implemented by the compiler for

* [Numeric types]
* `char`, `bool`, and [`!`]
* [Tuples] of `Copy` types
* [Arrays] of `Copy` types
* [Shared references]
* [Raw pointers]
* [Function pointers] and [function item types]

## `Clone`

The [`Clone`] trait is a supertrait of `Copy`, so it also needs compiler
generated implementations. It is implemented by the compiler for the following
types:

* Types with a built-in `Copy` implementation (see above)
* [Tuples] of `Clone` types
* [Arrays] of `Clone` types

## `Send`

The [`Send`] trait indicates that a value of this type is safe to send from one
thread to another.

## `Sync`

The [`Sync`] trait indicates that a value of this type is safe to share between
multiple threads. This trait must be implemented for all types used in
immutable [`static` items].

## Auto traits

The [`Send`], [`Sync`], [`Unpin`], [`UnwindSafe`], and [`RefUnwindSafe`] traits are _auto
traits_. Auto traits have special properties.

If no explicit implementation or negative implementation is written out for an
auto trait for a given type, then the compiler implements it automatically
according to the following rules:

* `&T`, `&mut T`, `*const T`, `*mut T`, `[T; n]`, and `[T]` implement the trait
  if `T` does.
* Function item types and function pointers automatically implement the trait.
* Structs, enums, unions, and tuples implement the trait if all of their fields
  do.
* Closures implement the trait if the types of all of their captures do. A
  closure that captures a `T` by shared reference and a `U` by value implements
  any auto traits that both `&T` and `U` do.

For generic types (counting the built-in types above as generic over `T`), if a
generic implementation is available, then the compiler does not automatically
implement it for types that could use the implementation except that they do not
meet the requisite trait bounds. For instance, the standard library implements
`Send` for all `&T` where `T` is `Sync`; this means that the compiler will not
implement `Send` for `&T` if `T` is `Send` but not `Sync`.

Auto traits can also have negative implementations, shown as `impl !AutoTrait
for T` in the standard library documentation, that override the automatic
implementations. For example `*mut T` has a negative implementation of `Send`,
and so `*mut T` is not `Send`, even if `T` is. There is currently no stable way
to specify additional negative implementations; they exist only in the standard
library.

Auto traits may be added as an additional bound to any [trait object], even
though normally only one trait is allowed. For instance, `Box<dyn Debug + Send +
UnwindSafe>` is a valid type.

## `Sized`

The [`Sized`] trait indicates that the size of this type is known at
compile-time; that is, it's not a [dynamically sized type]. [Type parameters]
are `Sized` by default. `Sized` is always implemented automatically by the
compiler, not by [implementation items].

[`Arc<Self>`]: ../std/sync/struct.Arc.html
[`Box<T>`]: ../std/boxed/struct.Box.html
[`Clone`]: ../std/clone/trait.Clone.html
[`Copy`]: ../std/marker/trait.Copy.html
[`Deref`]: ../std/ops/trait.Deref.html
[`DerefMut`]: ../std/ops/trait.DerefMut.html
[`Drop`]: ../std/ops/trait.Drop.html
[`Pin<P>`]: ../std/pin/struct.Pin.html
[`Rc<Self>`]: ../std/rc/struct.Rc.html
[`RefUnwindSafe`]: ../std/panic/trait.RefUnwindSafe.html
[`Send`]: ../std/marker/trait.Send.html
[`Sized`]: ../std/marker/trait.Sized.html
[`std::cell::UnsafeCell<T>`]: ../std/cell/struct.UnsafeCell.html
[`std::cmp`]: ../std/cmp/index.html
[`std::marker::PhantomData<T>`]: ../std/marker/struct.PhantomData.html
[`std::ops`]: ../std/ops/index.html
[`UnwindSafe`]: ../std/panic/trait.UnwindSafe.html
[`Sync`]: ../std/marker/trait.Sync.html
[`Unpin`]: ../std/marker/trait.Unpin.html

[Arrays]: types/array.md
[call expressions]: expressions/call-expr.md
[deref coercions]: type-coercions.md#coercion-types
[dereference operator]: expressions/operator-expr.md#the-dereference-operator
[destructor]: destructors.md
[drop check]: ../nomicon/dropck.html
[dynamically sized type]: dynamically-sized-types.md
[Function pointers]: types/function-pointer.md
[function item types]: types/function-item.md
[implementation items]: items/implementations.md
[indexing expressions]: expressions/array-expr.md#array-and-slice-indexing-expressions
[interior mutability]: interior-mutability.md
[Numeric types]: types/numeric.md
[Methods]: items/associated-items.md#associated-functions-and-methods
[method resolution]: expressions/method-call-expr.md
[operators]: expressions/operator-expr.md
[orphan rules]: items/implementations.md#trait实现的一致性
[Raw pointers]: types/pointer.md#raw-pointers-const-and-mut
[`static` items]: items/static-items.md
[Shared references]: types/pointer.md#shared-references-
[the standard library]: ../std/index.html
[trait object]: types/trait-object.md
[Tuples]: types/tuple.md
[Type parameters]: types/parameters.md
[variance]: subtyping.md#variance
[`!`]: types/never.md
