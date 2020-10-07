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

如果一个表达式是这些自动强转点中的一个，并且该表达式是传播自动强转的表达式(coercion-propagating expression)，那么该表达式中的相关子表达式也是自动强转点。传播从这些新的自动强转点递归。传播表达式(propagating expressions)及其相关子表达式有：

* 数组字面量，其中数组的类型为 `[U; n]`。数组字面量中的每个子表达式都是自动强转到类型 `U` 的自动强转点。

* 重复句法声明的数组字面量，其中数组的类型为 `[U; n]`。重复子表达式是用于自动强转到类型 `U` 的自动强转点。

* 元组，其中如果元组是自动强转到类型 `(U_0, U_1, ..., U_n)` 的强转点，则每个子表达式都是相应类型的自动强转点，比如第0个子表达式是到类型 `U_0`的 自动强转点。

* 圆括号括起来的子表达式(`(e)`)：如果整个表达式的类型为 `U`，则子表达式 `e` 是自动强转到类型 `U` 的自动强转点。

* 块：如果块的类型是 `U`，那么块中的最后一个表达式(如果它不是以分号结尾的)就是一个自动强转到类型 `U` 的自动强转点。这里的块包括作为控制流语句的一部分的条件分支代码块，比如 `if`/`else`，当然前提是这些块的返回需要有一个已知的类型。

## Coercion types
## 自动强转类型

自动强转允许发生在下列类型之间：

* `T` 到 `U` 如果 `T` 是 `U` 的一个[子类型][subtype] (*反射性场景(reflexive case)*)

* `T_1` 到 `T_3` 当 `T_1` 可自动强转到 `T_2` 同时 `T_2` 又能自动强转到 `T_3` (*传递性场景(transitive case)*)
    注意这个还没有得到完全支持。

* `&mut T` 到 `&T`

* `*mut T` 到 `*const T`

* `&T` 到 `*const T`

* `&mut T` 到 `*mut T`

* `&T` 或 `&mut T` 到 `&U` 如果 `T` 实现了 `Deref<Target = U>`。例如：

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
      foo(x); //&mut CharContainer 自动强转成 &char.
  }
  ```

* `&mut T` 到 `&mut U` 如果 `T` 实现 `DerefMut<Target = U>`.

* TyCtor(`T`) 到 TyCtor(`U`)，其中 TyCtor(`T`) 是下列之一(译者注：TyCtor为类型构造器 type constructor 的简写)
    - `&T`
    - `&mut T`
    - `*const T`
    - `*mut T`
    - `Box<T>`

    并且 `U` 能够通过[非固定尺寸类型自动强转](#unsized-coercions)得到。

    <!--In the future, coerce_inner will be recursively extended to tuples and
    structs. In addition, coercions from sub-traits to super-traits will be
    added. See [RFC 401] for more details.-->

* 非捕获闭包(Non capturing closures)到函数指针(`fn` pointers)

* `!` 到任意 `T`

### Unsized Coercions
### 非固定尺寸类型自动强转

下列自动强转被称为非固定尺寸类型自动强转(`unsized coercions`)，因为它们与*将固定尺寸类型转换为非固定尺寸类型*有关，并且在一些其他自动强转不允许的情况（也就是上节罗列的情况之外的情况）下允许使用。也就是他们可以发生在任何自动强转点，甚至自动强转点之外。

两个 trait，[`Unsize`] 和 [`CoerceUnsized`]，被用来协助这个转换发生，并供标准库来使用。以下胁迫内置模板,如果T可以强迫U与其中一个,然后一个实现Unsize < U > T将提供
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
