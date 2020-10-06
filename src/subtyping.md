# Subtyping and Variance
# 子类型化和型变

>[subtyping.md](https://github.com/rust-lang/reference/blob/master/src/subtyping.md)\
>commit ecb53d2015ce93b3519ee0b358fc13fa9b3f723d

子类型化是隐式的，可以出现在类型检查或推断的任何阶段。Rust 中的子类型化是非常受限制的，并且只发生在适配生存期和带高阶生存期的类型时。如果我们要擦除类型的生存期，那么唯一的子类型化将是由于类型相等。If we were to erase lifetimes from types, then the only subtyping would be due to type equality.

考虑下面的例子：字符串字面量总是拥有 `'static`生存期。不过，我们可以把 `s` 赋值给 `t`：

```rust
fn bar<'a>() {
    let s: &'static str = "hi";
    let t: &'a str = s;
}
```

因为 `'static` 比生存期参数 `'a` 的寿命长，所以 `&'static str` 是 `&'a str` 的子类型。

高阶函数指针和 trait对象是另一种子类型关系。它们是类型的子类型，这些类型是通过替换更高级别的生存期而给出的。一些例子
[Higher-ranked]&#32;[function pointers] and [trait objects] have another
subtype relation. They are subtypes of types that are given by substitutions of
the higher-ranked lifetimes. Some examples:

```rust
// Here 'a is substituted for 'static
let subtype: &(for<'a> fn(&'a i32) -> &'a i32) = &((|x| x) as fn(&_) -> &_);
let supertype: &(fn(&'static i32) -> &'static i32) = subtype;

// This works similarly for trait objects
let subtype: &(for<'a> Fn(&'a i32) -> &'a i32) = &|x| x;
let supertype: &(Fn(&'static i32) -> &'static i32) = subtype;

// We can also substitute one higher-ranked lifetime for another
let subtype: &(for<'a, 'b> fn(&'a i32, &'b i32))= &((|x, y| {}) as fn(&_, &_));
let supertype: &for<'c> fn(&'c i32, &'c i32) = subtype;
```

## Variance

Variance is a property that generic types have with respect to their arguments.
A generic type's *variance* in a parameter is how the subtyping of the
parameter affects the subtyping of the type.

* `F<T>` is *covariant* over `T` if `T` being a subtype of `U` implies that
  `F<T>` is a subtype of `F<U>` (subtyping "passes through")
* `F<T>` is *contravariant* over `T` if `T` being a subtype of `U` implies that
  `F<U>` is a subtype of `F<T>`
* `F<T>` is *invariant* over `T` otherwise (no subtyping relation can be
  derived)

Variance of types is automatically determined as follows

| Type                          | Variance in `'a`  | Variance in `T`   |
|-------------------------------|-------------------|-------------------|
| `&'a T`                       | covariant         | covariant         |
| `&'a mut T`                   | covariant         | invariant         |
| `*const T`                    |                   | covariant         |
| `*mut T`                      |                   | invariant         |
| `[T]` and `[T; n]`            |                   | covariant         |
| `fn() -> T`                   |                   | covariant         |
| `fn(T) -> ()`                 |                   | contravariant     |
| `std::cell::UnsafeCell<T>`    |                   | invariant         |
| `std::marker::PhantomData<T>` |                   | covariant         |
| `dyn Trait<T> + 'a`           | covariant         | invariant         |

The variance of other `struct`, `enum`, `union`, and tuple types is decided by
looking at the variance of the types of their fields. If the parameter is used
in positions with different variances then the parameter is invariant. For
example the following struct is covariant in `'a` and `T` and invariant in `'b`
and `U`.

```rust
use std::cell::UnsafeCell;
struct Variance<'a, 'b, T, U: 'a> {
    x: &'a U,               // This makes `Variance` covariant in 'a, and would
                            // make it covariant in U, but U is used later
    y: *const T,            // Covariant in T
    z: UnsafeCell<&'b f64>, // Invariant in 'b
    w: *mut U,              // Invariant in U, makes the whole struct invariant
}
```

[function pointers]: types/function-pointer.md
[Higher-ranked]: ../nomicon/hrtb.html
[trait objects]: types/trait-object.md
