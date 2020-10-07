# Type coercions
# 类型自动强转

>[type-coercions.md](https://github.com/rust-lang/reference/blob/master/src/type-coercions.md)\
>commit d5372c951cc37d76dc5d01e91ebfafdb30e3f50d

**类型自动强转**是改变值的类型的隐式操作。它们在特定的位置自动发生，并且在实际自动强转的类型上受到很多限制。

任何允许自动强转的转换都可以由[类型强制转换操作符][type cast operator] `as` 来显式执行。

自动强转最初是在 [RFC 401] 中定义的，并在[ RFC 1558] 中进行了扩展。

## Coercion sites
## 自动强转点

自动强转只能发生在程序中的某些自动强转点上；通常在这些位置上，所需的类型是显式给出了或者可以从显式类型的传播推导(be derived by propagation)得到(不是类型推断)。可能的强制点有：

* `let`语句中给出显式的类型。

   例如，下面例子中 `&mut 42` 自动强转成 `&i8` 类型：

   ```rust
   let _: &i8 = &mut 42;
   ```

* 静态(`static`)和常量(`const`)项声明（类似于 `let`语句）。

* 函数调用的参数

  被强制的值是实参(actual parameter)，它的类型被自动强转为形参(formal parameter)的类型。

  例如，下面例子中 `&mut 42` 自动强转成 `&i8` 类型：

  ```rust
  fn bar(_: &i8) { }

  fn main() {
      bar(&mut 42);
  }
  ```

  对于方法调用，接收者(`self`参数)只能使用[非固定尺寸类型自动强转(unsized coercion)](#unsized-coercions)。

* 实例化结构体、联合体或枚举变体的字段。

  例如，下面例子中 `&mut 42` 自动强转成 `&i8` 类型：

  ```rust
  struct Foo<'a> { x: &'a i8 }

  fn main() {
      Foo { x: &mut 42 };
  }
  ```
 
* 函数返回&mdash;函数体的最后一行如果不是以分号结尾的，函数将返回它的最后一行，或者是 `return`语句中的任何表达式

  例如，下面例子中 `x` 自动强转成 `&dyn Display` 类型：

  ```rust
  use std::fmt::Display;
  fn foo(x: &u32) -> &dyn Display {
      x
  }
  ```

如果其中一个强制点中的表达式是能传播自动强转的表达式(coercion-propagating expression)，那么该表达式中的相关子表达式也是自动强转点。传播从这些新的自动强转点递归。传播表达式(propagating expressions)及其相关子表达式有：
If the expression in one of these coercion sites is a coercion-propagating expression, then the relevant sub-expressions in that expression are also coercion sites. Propagation recurses from these new coercion sites. Propagating expressions and their relevant sub-expressions are:

* 数组字面量，其中数组的类型为 `[U; n]`。数组字面量中的每个子表达式都是用于自动强转到类型 `U` 的自动强转点。

* Array literals with repeating syntax, where the array has type `[U; n]`. The
repeated sub-expression is a coercion site for coercion to type `U`.

* Tuples, where a tuple is a coercion site to type `(U_0, U_1, ..., U_n)`.
Each sub-expression is a coercion site to the respective type, e.g. the
zeroth sub-expression is a coercion site to type `U_0`.

* Parenthesized sub-expressions (`(e)`): if the expression has type `U`, then
the sub-expression is a coercion site to `U`.

* Blocks: if a block has type `U`, then the last expression in the block (if
it is not semicolon-terminated) is a coercion site to `U`. This includes
blocks which are part of control flow statements, such as `if`/`else`, if
the block has a known type.

## Coercion types

Coercion is allowed between the following types:

* `T` to `U` if `T` is a [subtype] of `U` (*reflexive case*)

* `T_1` to `T_3` where `T_1` coerces to `T_2` and `T_2` coerces to `T_3`
(*transitive case*)

    Note that this is not fully supported yet.

* `&mut T` to `&T`

* `*mut T` to `*const T`

* `&T` to `*const T`

* `&mut T` to `*mut T`

* `&T` or `&mut T` to `&U` if `T` implements `Deref<Target = U>`. For example:

  ```rust
  use std::ops::Deref;

  struct CharContainer {
      value: char,
  }

  impl Deref for CharContainer {
      type Target = char;

      fn deref<'a>(&'a self) -> &'a char {
          &self.value
      }
  }

  fn foo(arg: &char) {}

  fn main() {
      let x = &mut CharContainer { value: 'y' };
      foo(x); //&mut CharContainer is coerced to &char.
  }
  ```

* `&mut T` to `&mut U` if `T` implements `DerefMut<Target = U>`.

* TyCtor(`T`) to TyCtor(`U`), where TyCtor(`T`) is one of
    - `&T`
    - `&mut T`
    - `*const T`
    - `*mut T`
    - `Box<T>`

    and where `U` can be obtained from `T` by [unsized coercion](#unsized-coercions).

    <!--In the future, coerce_inner will be recursively extended to tuples and
    structs. In addition, coercions from sub-traits to super-traits will be
    added. See [RFC 401] for more details.-->

* Non capturing closures to `fn` pointers

* `!` to any `T`

### Unsized Coercions

The following coercions are called `unsized coercions`, since they
relate to converting sized types to unsized types, and are permitted in a few
cases where other coercions are not, as described above. They can still happen
anywhere else a coercion can occur.

Two traits, [`Unsize`] and [`CoerceUnsized`], are used
to assist in this process and expose it for library use. The following
coercions are built-ins and, if `T` can be coerced to `U` with one of them, then
an implementation of `Unsize<U>` for `T` will be provided:

* `[T; n]` to `[T]`.

* `T` to `dyn U`, when `T` implements `U + Sized`, and `U` is [object safe].

* `Foo<..., T, ...>` to `Foo<..., U, ...>`, when:
    * `Foo` is a struct.
    * `T` implements `Unsize<U>`.
    * The last field of `Foo` has a type involving `T`.
    * If that field has type `Bar<T>`, then `Bar<T>` implements `Unsized<Bar<U>>`.
    * T is not part of the type of any other fields.

Additionally, a type `Foo<T>` can implement `CoerceUnsized<Foo<U>>` when `T`
implements `Unsize<U>` or `CoerceUnsized<Foo<U>>`. This allows it to provide a
unsized coercion to `Foo<U>`.

> Note: While the definition of the unsized coercions and their implementation
> has been stabilized, the traits themselves are not yet stable and therefore
> can't be used directly in stable Rust.

[RFC 401]: https://github.com/rust-lang/rfcs/blob/master/text/0401-coercions.md
[RFC 1558]: https://github.com/rust-lang/rfcs/blob/master/text/1558-closure-to-fn-coercion.md
[subtype]: subtyping.md
[object safe]: items/traits.md#object-safety
[type cast operator]: expressions/operator-expr.md#type-cast-expressions
[`Unsize`]: ../std/marker/trait.Unsize.html
[`CoerceUnsized`]: ../std/ops/trait.CoerceUnsized.html
