# Special types and traits
# 特殊类型和 trait

>[special-types-and-traits.md](https://github.com/rust-lang/reference/blob/master/src/special-types-and-traits.md)\
>commit: fcdc0cab546c10921d66054be25c6afc9dd6b3bc

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
## 运算符/操作符trait

[`std::ops`] 和 [`std::cmp`] 中的trait用于重载[运算符/操作符][operators]、[索引表达式][indexing expressions]和[调用表达式][call expressions]。

## `Deref` and `DerefMut`
## `Deref` 和 `DerefMut`

除了重载一元 `*`运算符外，[`Deref`] 和 [`DerefMut`] 也用于[方法解析(method resolution)][method resolution]和[利用`Deref`达成自动强转][deref coercions]。

## `Drop`

[`Drop`] trait 提供了一个[析构函数][destructor]，每当要销毁此类值时就会运行它。

## `Copy`

[`Copy`] trait 改变实现它的类型的语义。其类型实现 `Copy` 的值将在赋值时被复制而不是移动。

只能为未实现 `Drop` trait 且字段都是 `Copy` 的类型实现 `Copy`。
对于枚举，这意味着所有变体的所有字段都必须是 `Copy`。
对于联合体，这意味着所有的变体都必须是 `Copy`。

`Copy` 由编译器已实现给下述类型：

* [数字类类型][Numeric types]
* `char`, `bool`, 和 [`!`]
* 由 `Copy` 类型组成的[元组][Tuples]
* 由 `Copy` 类型组成的[数组][Arrays]
* [共享引用][Shared references]
* [裸指针][Raw pointers]
* [函数指针][Function pointers] 和 [函数项类型][function item types]

## `Clone`

[`Clone`] trait 是 `Copy` 的超类trait，所以它也需要编译器生成实现。它被编译器实现给了以下类型：

* 实现了内置的 `Copy` trait 的类型(见上面)
* 由 `Clone` 类型组成的[元组][Tuples]
* 由 `Clone` 类型组成的[数组][Arrays]

## `Send`

[`Send`] trait 表明这种类型的值可以安全地从一个线程发送到另一个线程。

## `Sync`

[`Sync`] trait 表示在多个线程之间共享这种类型的值是安全的。必须为不可变[静态(`static`)项][`static` items]中使用的所有类型实现此 trait。

## Auto traits

[`Send`]，[`Sync`]，[`Unpin`]，[`UnwindSafe`]，和 [`RefUnwindSafe`] trait 都是*自动trait*。自动trait 具有特殊的属性。

如果对于给定类型的自动trait 没有显式实现或否定实现(negative implementation)，那么编译器会根据以下规则自动此为类型去实现这些自动trait：

* 如果 `T` 实现了自动trait，那 `&T`, `&mut T`, `*const T`, `*mut T`, `[T; n]`, 和 `[T]` 也会实现。
* 函数项类型和函数指针自动实现这些 trait。
* 如果结构体、枚举、联合体和元组的所有字段都实现了这些 trait，则它们本身也会自动实现这些 trait。
* 如果闭包捕获的所有类型都实现了这些 trait，那么闭包会自动实现这些 trait。一个闭包通过共享引用捕获了`T`，同时通过所传值的方式捕获 `U`，那么该闭包会自动实现 `&T` 和 `U` 所共同实现的自动trait。

对于泛型类型(上面的这些内置类型也算是建立在 `T` 上的泛型)，如果泛型实现在当前已够用，则编译器不会为其再实现其他的自动trait，除非它们不满足必需的 trait约束。例如，在标准库里，在 `T` 实现了 `Sync` 的地方，那库就为所有 `&T` 实现了 `Send`；这意味着如果 `T` 是 `Send`，而不是 `Sync`，则编译器将不会为 `&T` 实现 `Send`。

自动trait 也可以有否定实现，在标准库文档中显示为 `impl !AutoTrait for T`，它覆盖了自动实现。例如，`*mut T` 有一个关于 `Send` 的否定实现，所以 `*mut T` 不是 `Send` 的，即使 `T` 是。目前在于标准库外还没有稳定的方法来指定额外的负面实现。

自动trait 可以附加到任何 [trait对象][trait object]上（通常我们见到的 trait对象的类型名上一般只允许显示一个trait）。例如，`Box<dyn Debug + Send + UnwindSafe>` 是一个有效的类型。

## `Sized`

[`Sized`] trait表明这种类型的尺寸在编译时是已知的；也就是说，它不是一个[动态尺寸类型][dynamically sized type]。[类型参数][Type parameters]默认是 `Sized` 的。`Sized` 总是由编译器自动实现，而不是由[实现项(implementation items)][implementation items]主动实现。

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
