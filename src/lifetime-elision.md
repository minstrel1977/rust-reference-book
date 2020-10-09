# Lifetime elision
# 生存期省略

>[destructors.md](https://github.com/rust-lang/reference/blob/master/src/destructors.md)\
>commit f8e76ee9368f498f7f044c719de68c7d95da9972

Rust 有一套在多个位置都允许省略生存期的规则，但要求在这些位置上编译器能推断出合理的默认生存期。

## Lifetime elision in functions
## 函数上的生存期省略

为了使常用模式更符合人体工程学，可以在[函数项][function item]、[函数指针][function pointer]和[闭包trait]签名中*省略*生存期参数。以下规则用于推断出被省略的生存期参数。省略不能推断出的生存期参数是错误的。占位符形式的生存期，`'_`，也可以用同样的方法来推断。对于路径中的生存期，首选使用 `'_`。trait对象的生存期遵循不同的规则，具体[这里](#default-trait-object-lifetimes)讨论。

* 参数中省略的每个生存期都会成为一个独立的生存期参数。
* 如果参数中使用了一个生存期(省略或不省略)，则将该生存期分配给*所有*省略了生存期参数的输出。

在方法签名中还有另一条规则

* 如果接收者有类型 `&Self` 或 `&mut Self`，那么对 `Self` 的引用的生存期会被分配给所有省略了生存期参数的输出。

示例：

```rust
# trait T {}
# trait ToCStr {}
# struct Thing<'a> {f: &'a i32}
# struct Command;
#
# trait Example {
fn print1(s: &str);                                   // 省略
fn print2(s: &'_ str);                                // 也省略
fn print3<'a>(s: &'a str);                            // 未省略

fn debug1(lvl: usize, s: &str);                       // 省略
fn debug2<'a>(lvl: usize, s: &'a str);                // 未省略

fn substr1(s: &str, until: usize) -> &str;            // 省略
fn substr2<'a>(s: &'a str, until: usize) -> &'a str;  // 未省略

fn get_mut1(&mut self) -> &mut dyn T;                 // 省略
fn get_mut2<'a>(&'a mut self) -> &'a mut dyn T;       // 未省略

fn args1<T: ToCStr>(&mut self, args: &[T]) -> &mut Command;                  // 省略
fn args2<'a, 'b, T: ToCStr>(&'a mut self, args: &'b [T]) -> &'a mut Command; // 未省略

fn new1(buf: &mut [u8]) -> Thing<'_>;                 // 省略 - 首选的
fn new2(buf: &mut [u8]) -> Thing;                     // 省略
fn new3<'a>(buf: &'a mut [u8]) -> Thing<'a>;          // 未省略
# }

type FunPtr1 = fn(&str) -> &str;                      // 省略
type FunPtr2 = for<'a> fn(&'a str) -> &'a str;        // 未省略

type FunTrait1 = dyn Fn(&str) -> &str;                // 省略
type FunTrait2 = dyn for<'a> Fn(&'a str) -> &'a str;  // 未省略
```

```rust,compile_fail
// 下面的示例显示了不允许省略生存期参数的情况。

# trait Example {
// 无法推断，因为没有可以推断的起始参数。
fn get_str() -> &str;                                 // 非法

// 无法推断，这里无法确认输出的生存期参数该遵从从第一个还是第二个参数的。
fn frob(s: &str, t: &str) -> &str;                    // 非法
# }
```

## Default trait object lifetimes
## 默认 trait对象的生存期

一个 [trait对象][trait object]所持有的引用的假定生存期(assumed lifetime)称为它的默认*对象生存期约束*。这些在 [RFC 599] 中定义，在 [RFC 1156] 中修定增补。

当生存期约束被完全省略时，会使用这些默认的对象生存期约束，而不是上面定义的生存期参数省略规则。但如果使用 `'_` 作为生存期约束，则该约束遵循通常的省略规则。

如果将 trait对象类型用作泛型类型的类型参数，则首先使用包含的类型来尝试推断一个约束。
If the trait object is used as a type argument of a generic type then the containing type is first used to try to infer a bound.

* 如果存在来自包含类型的唯一约束，则该约束为默认约束
* If there is a unique bound from the containing type then that is the default
* 如果包含类型有多个约束，则必须指定显式约束
* If there is more than one bound from the containing type then an explicit bound must be specified

如果这两个规则都不适用，则使用该 trait 的下述约束：
If neither of those rules apply, then the bounds on the trait are used:

* 如果 trait 定义为单个生命周期*约束*，则使用该约束。
* If the trait is defined with a single lifetime _bound_ then that bound is used.
* 如果 `'static` 用于任何生存期约束，则使用 `'static`。
* If `'static` is used for any lifetime bound then `'static` is used.
* 如果 trait没有生存期约束，那么生存期在表达式中被推断出来，在表达式之外是  `'static`。
* If the trait has no lifetime bounds, then the lifetime is inferred in expressions and is `'static` outside of expressions.

```rust
// 对下面 trait 来说，...
trait Foo { }

// 这两个和Box<T>是一样的对T没有生存期约束These two are the same as Box<T> has no lifetime bound on T
type T1 = Box<dyn Foo>;
type T2 = Box<dyn Foo + 'static>;

// ...and so are these:
impl dyn Foo {}
impl dyn Foo + 'static {}

// ...so are these, because &'a T requires T: 'a
type T3<'a> = &'a dyn Foo;
type T4<'a> = &'a (dyn Foo + 'a);

// std::cell::Ref<'a, T> also requires T: 'a, so these are the same
type T5<'a> = std::cell::Ref<'a, dyn Foo>;
type T6<'a> = std::cell::Ref<'a, dyn Foo + 'a>;
```

```rust,compile_fail
// This is an example of an error.
# trait Foo { }
struct TwoBounds<'a, 'b, T: ?Sized + 'a + 'b> {
    f1: &'a i32,
    f2: &'b i32,
    f3: T,
}
type T7<'a, 'b> = TwoBounds<'a, 'b, dyn Foo>;
//                                  ^^^^^^^
// Error: the lifetime bound for this object type cannot be deduced from context
```

Note that the innermost object sets the bound, so `&'a Box<dyn Foo>` is still
`&'a Box<dyn Foo + 'static>`.

```rust
// For the following trait...
trait Bar<'a>: 'a { }

// ...these two are the same:
type T1<'a> = Box<dyn Bar<'a>>;
type T2<'a> = Box<dyn Bar<'a> + 'a>;

// ...and so are these:
impl<'a> dyn Bar<'a> {}
impl<'a> dyn Bar<'a> + 'a {}
```

## `'static` lifetime elision

Both [constant] and [static] declarations of reference types have *implicit*
`'static` lifetimes unless an explicit lifetime is specified. As such, the
constant declarations involving `'static` above may be written without the
lifetimes.

```rust
// STRING: &'static str
const STRING: &str = "bitstring";

struct BitsNStrings<'a> {
    mybits: [u32; 2],
    mystring: &'a str,
}

// BITS_N_STRINGS: BitsNStrings<'static>
const BITS_N_STRINGS: BitsNStrings<'_> = BitsNStrings {
    mybits: [1, 2],
    mystring: STRING,
};
```

Note that if the `static` or `const` items include function or closure
references, which themselves include references, the compiler will first try
the standard elision rules. If it is unable to resolve the lifetimes by its
usual rules, then it will error. By way of example:

```rust
# struct Foo;
# struct Bar;
# struct Baz;
# fn somefunc(a: &Foo, b: &Bar, c: &Baz) -> usize {42}
// Resolved as `fn<'a>(&'a str) -> &'a str`.
const RESOLVED_SINGLE: fn(&str) -> &str = |x| x;

// Resolved as `Fn<'a, 'b, 'c>(&'a Foo, &'b Bar, &'c Baz) -> usize`.
const RESOLVED_MULTIPLE: &dyn Fn(&Foo, &Bar, &Baz) -> usize = &somefunc;
```

```rust,compile_fail
# struct Foo;
# struct Bar;
# struct Baz;
# fn somefunc<'a,'b>(a: &'a Foo, b: &'b Bar) -> &'a Baz {unimplemented!()}
// There is insufficient information to bound the return reference lifetime
// relative to the argument lifetimes, so this is an error.
const RESOLVED_STATIC: &dyn Fn(&Foo, &Bar) -> &Baz = &somefunc;
//                                            ^
// this function's return type contains a borrowed value, but the signature
// does not say whether it is borrowed from argument 1 or argument 2
```

[closure trait]: types/closure.md
[constant]: items/constant-items.md
[function item]: types/function-item.md
[function pointer]: types/function-pointer.md
[RFC 599]: https://github.com/rust-lang/rfcs/blob/master/text/0599-default-object-bound.md
[RFC 1156]: https://github.com/rust-lang/rfcs/blob/master/text/1156-adjust-default-object-bounds.md
[static]: items/static-items.md
[trait object]: types/trait-object.md
