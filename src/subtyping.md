# Subtyping and Variance
# 子类型化和型变

>[subtyping.md](https://github.com/rust-lang/reference/blob/master/src/subtyping.md)\
>commit: 36a66c0ccf9f8a5b941b0df9923ece80d8714c83 \
>本章译文最后维护日期：2022-08-21

子类型化是隐式的，可以出现在类型检查或类型推断的任何阶段。
子类型化仅限于两种情况：引入生存期(lifetimes)带来的类型型变(variance)，以及引入高阶生存期带来的类型型变之间。如果我们从类型中擦除了生存期，那么子类型化只能是类型相等(type equality)了。

考虑下面的例子：字符串字面量总是拥有 `'static`生存期。不过，我们还是可以把 `s` 赋值给 `t`：

```rust
fn bar<'a>() {
    let s: &'static str = "hi";
    let t: &'a str = s;
}
```

因为 `'static` 比生存期参数 `'a` 的寿命长，所以 `&'static str` 是 `&'a str` 的子类型。

[高阶][Higher-ranked][函数指针][function pointers]和[trait对象][trait objects]可以形成另一种父子类型的关系。
它们是那些通过替换高阶生存期而得出的类型的子类型。举些例子：

```rust
// 这里 'a 被替换成了 'static
let subtype: &(for<'a> fn(&'a i32) -> &'a i32) = &((|x| x) as fn(&_) -> &_);
let supertype: &(fn(&'static i32) -> &'static i32) = subtype;

// 这对于 trait对象也是类似的
let subtype: &(dyn for<'a> Fn(&'a i32) -> &'a i32) = &|x| x;
let supertype: &(dyn Fn(&'static i32) -> &'static i32) = subtype;

// 我们也可以用一个高阶生存期来代替另一个
let subtype: &(for<'a, 'b> fn(&'a i32, &'b i32))= &((|x, y| {}) as fn(&_, &_));
let supertype: &for<'c> fn(&'c i32, &'c i32) = subtype;
```

## Variance
## 型变

型变是泛型类型相对其参数具有的属性。
泛型类型在它的某个参数上的*型变*是描述该参数的子类型化去如何影响此泛型类型的子类型化。

* 如果 `T` 是 `U` 的一个子类型意味着 `F<T>` 是 `F<U>` 的一个子类型（即子类型化“通过(passes through)”），则 `F<T>` 在 `T` 上是*协变的(covariant)*。
* 如果 `T` 是 `U` 的一个子类型意味着 `F<U>` 是 `F<T>` 的一个子类型，则 `F<T>` 在 `T` 上是*逆变的(contravariant)*。
* 其他情况下（即不能由参数类型的子类型化关系推导出此泛型的型变关系），`F<T>` 在 `T` 上是的*不变的(invariant)*。

类型的型变关系由下表中的规则自动确定：

| Type                          | 在 `'a` 上的型变 |  在 `T` 上的型变   |
|-------------------------------|-------------------|-------------------|
| `&'a T`                       | 协变的         | 协变的         |
| `&'a mut T`                   | 协变的         | 不变的         |
| `*const T`                    |                   | 协变的         |
| `*mut T`                      |                   | 不变的         |
| `[T]` 和 `[T; n]`             |                   | 协变的         |
| `fn() -> T`                   |                   | 协变的         |
| `fn(T) -> ()`                 |                   | 逆变的     |
| `std::cell::UnsafeCell<T>`    |                   | 不变的         |
| `std::marker::PhantomData<T>` |                   | 协变的         |
| `dyn Trait<T> + 'a`           | 协变的         | 不变的         |

结构体(`struct`)、枚举(`enum`)和联合体(`union`)类型上的型变关系是通过查看其字段类型的型变关系来决定的。
如果参数用在了多处且具有不同型变关系的位置上，则该类型在该参数上是不变的。
例如，下面示例的结构体在 `'a` 和 `T` 上是协变的，在 `'b`、`'c` 和 `U` 上是不变的。

```rust
use std::cell::UnsafeCell;
struct Variance<'a, 'b, 'c, T, U: 'a> {
    x: &'a U,               // 这让 `Variance` 在 'a 上是协变的, 也让在 U 上是协变的, 但是后面也使用了 U
    y: *const T,            // 在 T 上是协变的
    z: UnsafeCell<&'b f64>, // 在 'b 上是不变的
    w: *mut U,              // 在 U 上是不变的, 所以让整个结构体在 U 上是不变的

    f: fn(&'c ()) -> &'c () // 无论是协变还是逆变，都会导致此结构体在 'c 上不变。
}
```

当在结构体(`struct`)、枚举(`enum`)和联合体(`union`)之外使用时，会在每个位置分别检查参数的型变。

```rust
# use std::cell::UnsafeCell;
fn generic_tuple<'short, 'long: 'short>(
    // 'long 同时应用在一个元组里的协变和不变的位置上
    x: (&'long u32, UnsafeCell<&'long u32>),
) {
    // 由于这些位置的型变是单独计算的，我们可以在协变位置自由地收缩 'long。
    let _: (&'short u32, UnsafeCell<&'long u32>) = x;
}

fn takes_fn_ptr<'short, 'middle: 'short>(
    // 'middle 被用在了一个既是协变又是逆变的位置上。
    f: fn(&'middle ()) -> &'middle (),
) {
    // 由于这些位置的型变是单独计算的，我们可以在协变位置自由地收缩 'middle，并在逆变位置拉伸它。
    let _: fn(&'static ()) -> &'short () = f;
}
```

[function pointers]: types/function-pointer.md
[Higher-ranked]: https://doc.rust-lang.org/nomicon/hrtb.html
[trait objects]: types/trait-object.md