# Type Layout
# 类型布局

>[type-layout.md](https://github.com/rust-lang/reference/blob/master/src/type-layout.md)\
>commit 66b4d58fc0830757a98be2c5ba390255168056f7

类型的布局描述类型的尺寸、对齐和字段的相对偏移量。对于枚举，如何布局和解释判别值也是类型布局的一部分。

每次编译都可以更改类型布局。这里我们只阐述当前编译器所保证的内容，而没试图去记录编译器对此做了什么。

## Size and Alignment
## 尺寸和对齐

所有值都有对齐和尺寸。

值的对齐指定了哪些地址可以有效地存储该值。对齐为n的值只能存储在n的倍数地址上。例如，对齐为2的值必须存储在偶数地址上，而对齐为1的值可以存储在任何地址上。对齐是用字节来度量的，必须至少是1，并且总是2的幂次。值的对齐可以通过 [`align_of_val`] 函数来检查。

值的*尺寸*是同类型的值组成的数组中连续元素之间的字节偏移量。值的尺寸总是其对齐的倍数。值的尺寸可以通过 [`size_of_val`] 函数来检查。

类型如果实现了 [`Sized`] trait，那它的所有值在编译时都具有相同大小和对齐，并且可以使用函数 [`size_of`] 和 [`align_of`] 对此类型进行检测。没有实现 `Sized` trait 的类型被称为[动态尺寸类型][dynamically sized types]。由于实现了 `Sized` trait某一类型的所有值共享相同的大小和对齐，所以我们分别将这俩共享值称为类型的大小和类型的对齐。

## Primitive Data Layout
## 原生数据类型的布局

下表给出了大多数原生数据类型(primitives)的尺寸。

| 类型              | `size_of::<Type>()`|
|--                 |--                  |
| `bool`            | 1                  |
| `u8` / `i8`       | 1                  |
| `u16` / `i16`     | 2                  |
| `u32` / `i32`     | 4                  |
| `u64` / `i64`     | 8                  |
| `u128` / `i128`   | 16                 |
| `f32`             | 4                  |
| `f64`             | 8                  |
| `char`            | 4                  |

`usize` 和 `isize` 的尺寸足以包含目标平台上的每个内存地址。例如，在32位目标上，这是4个字节，而在64位目标上，这是8个字节。

大多数原生数据类型通常与它们的尺寸保持一致，尽管这是特定于平台的行为。具体地，在x86平台上，u64 和 f64 都上32位对齐。

## Pointers and References Layout
## 指针和引用的布局

指针和引用具有相同的布局。指针或引用的可变性不会改变布局。

指向固定尺寸类型(sized type)的指针具有与 `usize` 相同的尺寸和对齐。

指向非固定尺寸类型的指针是固定尺寸的。其尺寸和对齐至少等于一个指针的尺寸和对齐

> 注意：虽然不应该依赖于此，但是目前所有指向 <abbr title="Dynamically Sized Types">DST</abbr> 的指针都是 `usize` 的两倍大小，并且具有相同的对齐方式。

## Array Layout
## 数组的布局

数组的布局使得数组的第n个(`nth`)元素为从数组开始的位置向后偏移 _n*元素类型的尺寸(`n * the size of the element's type`)_ 个字节数。数组 `[T; n]` 的尺寸为`size_of::<T>() * n`，对齐和 `T` 的对齐相同。

## Slice Layout
## 切片的布局

切片的布局与它们所切的那部分数组片段相同。

> 注意：这是关于原生的 `[T]`类型，而不是指向切片的指针`&[T]`、`Box<[T]>`、等等)。

## `str` Layout
## 字符串切片(`str`)的布局

字符串切片是与 `[u8]`类型的切片具有相同布局的字符序列的 UTF-8表示形式(representation)。

## Tuple Layout
## 元组的布局

元组对于其布局没有任何保证。

这方面的例外是单元结构体(unit tuple)(`()`)，它被保证为大小为0的类型，其大小为0，对齐方式为1
The exception to this is the unit tuple (`()`) which is guaranteed as a zero-sized type to have a size of 0 and an alignment of 1.

## Trait Object Layout

Trait objects have the same layout as the value the trait object is of.

> Note: This is about the raw trait object types, not pointers (`&dyn Trait`,
> `Box<dyn Trait>`, etc.) to trait objects.

## Closure Layout

Closures have no layout guarantees.

## Representations

All user-defined composite types (`struct`s, `enum`s, and `union`s) have a
*representation* that specifies what the layout is for the type. The possible
representations for a type are:

- [Default]
- [`C`]
- The [primitive representations]
- [`transparent`]

The representation of a type can be changed by applying the `repr` attribute
to it. The following example shows a struct with a `C` representation.

```rust
#[repr(C)]
struct ThreeInts {
    first: i16,
    second: i8,
    third: i32
}
```

The alignment may be raised or lowered with the `align` and `packed` modifiers
respectively. They alter the representation specified in the attribute.
If no representation is specified, the default one is altered.

```rust
// Default representation, alignment lowered to 2.
#[repr(packed(2))]
struct PackedStruct {
    first: i16,
    second: i8,
    third: i32
}

// C representation, alignment raised to 8
#[repr(C, align(8))]
struct AlignedStruct {
    first: i16,
    second: i8,
    third: i32
}
```

> Note: As a consequence of the representation being an attribute on the item,
> the representation does not depend on generic parameters. Any two types with
> the same name have the same representation. For example, `Foo<Bar>` and
> `Foo<Baz>` both have the same representation.

The representation of a type can change the padding between fields, but does
not change the layout of the fields themselves. For example, a struct with a
`C` representation that contains a struct `Inner` with the default
representation will not change the layout of `Inner`.

### The Default Representation

Nominal types without a `repr` attribute have the default representation.
Informally, this representation is also called the `rust` representation.

There are no guarantees of data layout made by this representation.

### The `C` Representation

The `C` representation is designed for dual purposes. One purpose is for
creating types that are interoperable with the C Language. The second purpose is
to create types that you can soundly perform operations on that rely on data
layout such as reinterpreting values as a different type.

Because of this dual purpose, it is possible to create types that are not useful
for interfacing with the C programming language.

This representation can be applied to structs, unions, and enums. The exception
is [zero-variant enums] for which the `C` representation is an error.

#### \#[repr(C)] Structs

The alignment of the struct is the alignment of the most-aligned field in it.

The size and offset of fields is determined by the following algorithm.

Start with a current offset of 0 bytes.

For each field in declaration order in the struct, first determine the size and
alignment of the field. If the current offset is not a multiple of the field's
alignment, then add padding bytes to the current offset until it is a multiple
of the field's alignment. The offset for the field is what the current offset
is now. Then increase the current offset by the size of the field.

Finally, the size of the struct is the current offset rounded up to the nearest
multiple of the struct's alignment.

Here is this algorithm described in pseudocode.

<!-- ignore: pseudocode -->
```rust,ignore
/// Returns the amount of padding needed after `offset` to ensure that the
/// following address will be aligned to `alignment`.
fn padding_needed_for(offset: usize, alignment: usize) -> usize {
    let misalignment = offset % alignment;
    if misalignment > 0 {
        // round up to next multiple of `alignment`
        alignment - misalignment
    } else {
        // already a multiple of `alignment`
        0
    }
}

struct.alignment = struct.fields().map(|field| field.alignment).max();

let current_offset = 0;

for field in struct.fields_in_declaration_order() {
    // Increase the current offset so that it's a multiple of the alignment
    // of this field. For the first field, this will always be zero.
    // The skipped bytes are called padding bytes.
    current_offset += padding_needed_for(current_offset, field.alignment);

    struct[field].offset = current_offset;

    current_offset += field.size;
}

struct.size = current_offset + padding_needed_for(current_offset, struct.alignment);
```

<div class="warning">

Warning: This pseudocode uses a naive algorithm that ignores overflow issues for
the sake of clarity. To perform memory layout computations in actual code, use
[`Layout`].

</div>

> Note: This algorithm can produce zero-sized structs. In C, an empty struct
> declaration like `struct Foo { }` is illegal. However, both gcc and clang
> support options to enable such structs, and assign them size zero. C++, in
> contrast, gives empty structs a size of 1, unless they are inherited from or
> they are fields that have the `[[no_unique_address]]` attribute, in which
> case they do not increase the overall size of the struct.

#### \#[repr(C)] Unions

A union declared with `#[repr(C)]` will have the same size and alignment as an
equivalent C union declaration in the C language for the target platform.
The union will have a size of the maximum size of all of its fields rounded to
its alignment, and an alignment of the maximum alignment of all of its fields.
These maximums may come from different fields.

```rust
#[repr(C)]
union Union {
    f1: u16,
    f2: [u8; 4],
}

assert_eq!(std::mem::size_of::<Union>(), 4);  // From f2
assert_eq!(std::mem::align_of::<Union>(), 2); // From f1

#[repr(C)]
union SizeRoundedUp {
   a: u32,
   b: [u16; 3],
}

assert_eq!(std::mem::size_of::<SizeRoundedUp>(), 8);  // Size of 6 from b,
                                                      // rounded up to 8 from
                                                      // alignment of a.
assert_eq!(std::mem::align_of::<SizeRoundedUp>(), 4); // From a
```

#### \#[repr(C)] Field-less Enums

For [field-less enums], the `C` representation has the size and alignment of
the default `enum` size and alignment for the target platform's C ABI.

> Note: The enum representation in C is implementation defined, so this is
> really a "best guess". In particular, this may be incorrect when the C code
> of interest is compiled with certain flags.

<div class="warning">

Warning: There are crucial differences between an `enum` in the C language and
Rust's [field-less enums] with this representation. An `enum` in C is
mostly a `typedef` plus some named constants; in other words, an object of an
`enum` type can hold any integer value. For example, this is often used for
bitflags in `C`. In contrast, Rust’s [field-less enums] can only legally hold
the discrimnant values, everything else is [undefined behavior]. Therefore,
using a field-less enum in FFI to model a C `enum` is often wrong.

</div>

#### \#[repr(C)] Enums With Fields

The representation of a `repr(C)` enum with fields is a `repr(C)` struct with
two fields, also called a "tagged union" in C:

- a `repr(C)` version of the enum with all fields removed ("the tag")
- a `repr(C)` union of `repr(C)` structs for the fields of each variant that had
  them ("the payload")

> Note: Due to the representation of `repr(C)` structs and unions, if a variant
> has a single field there is no difference between putting that field directly
> in the union or wrapping it in a struct; any system which wishes to manipulate
> such an `enum`'s representation may therefore use whichever form is more
> convenient or consistent for them.

```rust
// This Enum has the same representation as ...
#[repr(C)]
enum MyEnum {
    A(u32),
    B(f32, u64),
    C { x: u32, y: u8 },
    D,
 }

// ... this struct.
#[repr(C)]
struct MyEnumRepr {
    tag: MyEnumDiscriminant,
    payload: MyEnumFields,
}

// This is the discriminant enum.
#[repr(C)]
enum MyEnumDiscriminant { A, B, C, D }

// This is the variant union.
#[repr(C)]
union MyEnumFields {
    A: MyAFields,
    B: MyBFields,
    C: MyCFields,
    D: MyDFields,
}

#[repr(C)]
#[derive(Copy, Clone)]
struct MyAFields(u32);

#[repr(C)]
#[derive(Copy, Clone)]
struct MyBFields(f32, u64);

#[repr(C)]
#[derive(Copy, Clone)]
struct MyCFields { x: u32, y: u8 }

// This struct could be omitted (it is a zero-sized type), and it must be in
// C/C++ headers.
#[repr(C)]
#[derive(Copy, Clone)]
struct MyDFields;
```

> Note: `union`s with non-`Copy` fields are unstable, see [55149].

### Primitive representations

The *primitive representations* are the representations with the same names as
the primitive integer types. That is: `u8`, `u16`, `u32`, `u64`, `u128`,
`usize`, `i8`, `i16`, `i32`, `i64`, `i128`, and `isize`.

Primitive representations can only be applied to enumerations and have
different behavior whether the enum has fields or no fields. It is an error
for [zero-variant enumerations] to have a primitive representation. Combining
two primitive representations together is an error.

#### Primitive Representation of Field-less Enums

For [field-less enums], primitive representations set the size and alignment to
be the same as the primitive type of the same name. For example, a field-less
enum with a `u8` representation can only have discriminants between 0 and 255
inclusive.

#### Primitive Representation of Enums With Fields

The representation of a primitive representation enum is a `repr(C)` union of
`repr(C)` structs for each variant with a field. The first field of each struct
in the union is the primitive representation version of the enum with all fields
removed ("the tag") and the remaining fields are the fields of that variant.

> Note: This representation is unchanged if the tag is given its own member in
> the union, should that make manipulation more clear for you (although to
> follow the C++ standard the tag member should be wrapped in a `struct`).

```rust
// This enum has the same representation as ...
#[repr(u8)]
enum MyEnum {
    A(u32),
    B(f32, u64),
    C { x: u32, y: u8 },
    D,
 }

// ... this union.
#[repr(C)]
union MyEnumRepr {
    A: MyVariantA,
    B: MyVariantB,
    C: MyVariantC,
    D: MyVariantD,
}

// This is the discriminant enum.
#[repr(u8)]
#[derive(Copy, Clone)]
enum MyEnumDiscriminant { A, B, C, D }

#[repr(C)]
#[derive(Clone, Copy)]
struct MyVariantA(MyEnumDiscriminant, u32);

#[repr(C)]
#[derive(Clone, Copy)]
struct MyVariantB(MyEnumDiscriminant, f32, u64);

#[repr(C)]
#[derive(Clone, Copy)]
struct MyVariantC { tag: MyEnumDiscriminant, x: u32, y: u8 }

#[repr(C)]
#[derive(Clone, Copy)]
struct MyVariantD(MyEnumDiscriminant);
```

> Note: `union`s with non-`Copy` fields are unstable, see [55149].

#### Combining primitive representations of enums with fields and \#[repr(C)]

For enums with fields, it is also possible to combine `repr(C)` and a
primitive representation (e.g., `repr(C, u8)`). This modifies the [`repr(C)`] by
changing the representation of the discriminant enum to the chosen primitive
instead. So, if you chose the `u8` representation, then the discriminant enum
would have a size and alignment of 1 byte.

The discriminant enum from the example [earlier][`repr(C)`] then becomes:

```rust
#[repr(C, u8)] // `u8` was added
enum MyEnum {
    A(u32),
    B(f32, u64),
    C { x: u32, y: u8 },
    D,
 }

// ...

#[repr(u8)] // So `u8` is used here instead of `C`
enum MyEnumDiscriminant { A, B, C, D }

// ...
```

For example, with a `repr(C, u8)` enum it is not possible to have 257 unique
discriminants ("tags") whereas the same enum with only a `repr(C)` attribute
will compile without any problems.

Using a primitive representation in addition to `repr(C)` can change the size of
an enum from the `repr(C)` form:

```rust
#[repr(C)]
enum EnumC {
    Variant0(u8),
    Variant1,
}

#[repr(C, u8)]
enum Enum8 {
    Variant0(u8),
    Variant1,
}

#[repr(C, u16)]
enum Enum16 {
    Variant0(u8),
    Variant1,
}

// The size of the C representation is platform dependant
assert_eq!(std::mem::size_of::<EnumC>(), 8);
// One byte for the discriminant and one byte for the value in Enum8::Variant0
assert_eq!(std::mem::size_of::<Enum8>(), 2);
// Two bytes for the discriminant and one byte for the value in Enum16::Variant0
// plus one byte of padding.
assert_eq!(std::mem::size_of::<Enum16>(), 4);
```

[`repr(C)`]: #reprc-enums-with-fields

### The alignment modifiers

The `align` and `packed` modifiers can be used to respectively raise or lower
the alignment of `struct`s and `union`s. `packed` may also alter the padding
between fields.

The alignment is specified as an integer parameter in the form of
`#[repr(align(x))]` or `#[repr(packed(x))]`. The alignment value must be a
power of two from 1 up to 2<sup>29</sup>. For `packed`, if no value is given,
as in `#[repr(packed)]`, then the value is 1.

For `align`, if the specified alignment is less than the alignment of the type
without the `align` modifier, then the alignment is unaffected.

For `packed`, if the specified alignment is greater than the type's alignment
without the `packed` modifier, then the alignment and layout is unaffected.
The alignments of each field, for the purpose of positioning fields, is the
smaller of the specified alignment and the alignment of the field's type.

The `align` and `packed` modifiers cannot be applied on the same type and a
`packed` type cannot transitively contain another `align`ed type. `align` and
`packed` may only be applied to the [default] and [`C`] representations.

The `align` modifier can also be applied on an `enum`.
When it is, the effect on the `enum`'s alignment is the same as if the `enum`
was wrapped in a newtype `struct` with the same `align` modifier.

<div class="warning">

***Warning:*** Dereferencing an unaligned pointer is [undefined behavior] and
it is possible to [safely create unaligned pointers to `packed` fields][27060].
Like all ways to create undefined behavior in safe Rust, this is a bug.

</div>

### The `transparent` Representation

The `transparent` representation can only be used on a [`struct`][structs]
or an [`enum`][enumerations] with a single variant that has:

- a single field with non-zero size, and
- any number of fields with size 0 and alignment 1 (e.g. [`PhantomData<T>`]).

Structs and enums with this representation have the same layout and ABI
as the single non-zero sized field.

This is different than the `C` representation because
a struct with the `C` representation will always have the ABI of a `C` `struct`
while, for example, a struct with the `transparent` representation with a
primitive field will have the ABI of the primitive field.

Because this representation delegates type layout to another type, it cannot be
used with any other representation.

[`align_of_val`]: ../std/mem/fn.align_of_val.html
[`size_of_val`]: ../std/mem/fn.size_of_val.html
[`align_of`]: ../std/mem/fn.align_of.html
[`size_of`]: ../std/mem/fn.size_of.html
[`Sized`]: ../std/marker/trait.Sized.html
[`Copy`]: ../std/marker/trait.Copy.html
[dynamically sized types]: dynamically-sized-types.md
[field-less enums]: items/enumerations.md#custom-discriminant-values-for-fieldless-enumerations
[enumerations]: items/enumerations.md
[zero-variant enums]: items/enumerations.md#zero-variant-enums
[undefined behavior]: behavior-considered-undefined.md
[27060]: https://github.com/rust-lang/rust/issues/27060
[55149]: https://github.com/rust-lang/rust/issues/55149
[`PhantomData<T>`]: special-types-and-traits.md#phantomdatat
[Default]: #the-default-representation
[`C`]: #the-c-representation
[primitive representations]: #primitive-representations
[structs]: items/structs.md
[`transparent`]: #the-transparent-representation
[`Layout`]: ../std/alloc/struct.Layout.html
