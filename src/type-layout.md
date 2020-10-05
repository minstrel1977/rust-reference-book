# Type Layout
# 类型布局

>[type-layout.md](https://github.com/rust-lang/reference/blob/master/src/type-layout.md)\
>commit 66b4d58fc0830757a98be2c5ba390255168056f7

类型的布局描述类型的尺寸(size)、对齐量(alignment)和字段(fields)的*相对偏移量(relative offsets)*。对于枚举，如何布局和解释判别值(discriminant)也是类型布局的一部分。

每次编译都可以更改类型布局。这里我们只阐述当前编译器所保证的内容，而没试图去记录编译器对此做了什么。

## Size and Alignment
## 尺寸和对齐量

所有值都有对齐量和尺寸。

值的对齐量指定了哪些地址可以有效地存储该值。对齐量为n的值只能存储在n的倍数地址上。例如，对齐量为2的值必须存储在偶数地址上，而对齐量为1的值可以存储在任何地址上。对齐量是用字节来度量的，必须至少是1，并且总是2的幂次。值的对齐量可以通过 [`align_of_val`] 函数来检查。

值的*尺寸*是同类型的值组成的数组中连续元素之间的字节偏移量。值的尺寸总是其对齐量的倍数。值的尺寸可以通过 [`size_of_val`] 函数来检查。

类型如果实现了 [`Sized`] trait，那它的所有值在编译时都具有相同尺寸和对齐量，并且可以使用函数 [`size_of`] 和 [`align_of`] 对此类型进行检测。没有实现 `Sized` trait 的类型被称为[动态尺寸类型][dynamically sized types]。由于实现了 `Sized` trait某一类型的所有值共享相同的尺寸和对齐量，所以我们分别将这俩共享值称为类型的尺寸和类型的对齐量。

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

大多数原生数据类型通常与它们的尺寸保持一致，尽管这是特定于平台的行为。具体地，在x86平台上，u64 和 f64 都上32位对齐量。

## Pointers and References Layout
## 指针和引用的布局

指针和引用具有相同的布局。指针或引用的可变性不会改变布局。

指向固定尺寸类型(sized type)的指针具有与 `usize` 相同的尺寸和对齐量。

指向非固定尺寸类型的指针是固定尺寸的。其尺寸和对齐量至少等于一个指针的尺寸和对齐量

> 注意：虽然不应该依赖于此，但是目前所有指向 <abbr title="Dynamically Sized Types">DST</abbr> 的指针都是 `usize` 的两倍尺寸，并且具有相同的对齐量方式。

## Array Layout
## 数组的布局

数组的布局使得数组的第n个(`nth`)元素为从数组开始的位置向后偏移 _n*元素类型的尺寸(`n * the size of the element's type`)_ 个字节数。数组 `[T; n]` 的尺寸为`size_of::<T>() * n`，对齐量和 `T` 的对齐量相同。

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

这方面的例外是单元结构体(unit tuple)(`()`)类型，它被保证为尺寸为0，对齐量为1。

## Trait Object Layout
## trait对象的布局

trait对象的布局与 trait对象的值相同。

> 注意：这是关于原生 trait对象类型(raw trait object type)的，而不是指向 trait对象的指针(`&dyn Trait`， `Box<dyn Trait>` 等)。

## Closure Layout
## 闭包的布局

闭包没有布局保证。

## Representations
## 表形/表示形式

所有用户定义的复合类型(结构体(`struct`)、枚举(`enum`)和联合体(`union`))都有一个*表形(representation)*属性，该属性用于指定该类型的布局。类型的可能表形是：

- [默认(default)表形][Default]
- [`C`表形][`C`]
- [原语表形(primitive representation)][primitive representations]
- [透明表形(`transparent`)][`transparent`]

类型的表形可以通过对其应用 `repr`属性来更改。下面的示例展示了一个 `C`表形的结构体。

```rust
#[repr(C)]
struct ThreeInts {
    first: i16,
    second: i8,
    third: i32
}
```

可以分别使用 `align` 和 `packed` 修饰符增大或缩小对齐量。它们可以更改属性中指定的表形的对齐量。如果未指定表形，则更改默认表形的。

```rust
// 默认表形，把对齐量缩小到2。
#[repr(packed(2))]
struct PackedStruct {
    first: i16,
    second: i8,
    third: i32
}

// C表形，把对齐量增大到8
#[repr(C, align(8))]
struct AlignedStruct {
    first: i16,
    second: i8,
    third: i32
}
```

> 注意：由于表形是数据项的属性，因此表形不依赖于泛型参数。具有相同名称的任何两种类型都具有相同的表形。例如，`Foo<Bar>` 和 `Foo<Baz>` 都有相同的表形。

类型的表形可以更改字段之间的填充，但不会更改字段本身的布局。例如一个使用 `C`表形的结构体，如果它包含一个默认表形的结构 `Inner`，那么它不会改变  `Inner` 的布局。

### The Default Representation
### 默认表形

没有 `repr`属性的标称(nominal)类型具有默认表形。非正式地的情况下，也称这种表形为 `rust`表形。

这种表形不能保证(两次编译具有相同的)数据布局。

### The `C` Representation
### `C`表形

`C`表形被设计用于双重目的：一个目的是创建可以与C语言互操作的类型；第二个目的是创建可以正确执行依赖于数据布局的操作的类型，比如将值重新解释为其他类型。

因为这种双重目的存在，可以只利用其中的一个目的只创建有固定布局的类型，而放弃与C语言互操作的目的。

这种表型可以应用于结构体(structs)、联合体(unions)和枚举(enums)。一个例外是[零变体枚举(zero-variant enums)][zero-variant enums]，它的`C`表形是错误的。

#### \#[repr(C)] Structs
#### \#[repr(C)] 结构体

结构体的对齐量是其*最大对齐量的字段(most-aligned field)*的对齐量。

字段的尺寸和偏移量由以下算法确定：

1. 从首字段开始的地址计为偏移量0字节开始。

2. 对于结构体中按顺序声明的每个字段，首先确定字段的尺寸和对齐量。如果当前偏移量不是字段对齐量的倍数，则向当前偏移量添加填充字节，直到它是字段对齐量的倍数。字段的偏移量是当前字段的开始地址相对于0字节的偏移量。然后在根据当前字段的尺寸增加当前偏移量。然后重复本逻辑，直至所有字段都完成。
   
3. 最后，结构体的尺寸是当前偏移量，向上取整到结构体对齐量的最近倍数。

下面用伪代码描述这个算法：

<!-- ignore: pseudocode -->
```rust,ignore
/// 返回偏移(`offset`)之后需要的填充量，以确保接下来的地址将被安排到可对齐的地址。
fn padding_needed_for(offset: usize, alignment: usize) -> usize {
    let misalignment = offset % alignment;
    if misalignment > 0 {
        // 向上取整到对齐量(`alignment`)的下一个倍数
        alignment - misalignment
    } else {
        // 已经是对齐量(`alignment`)的倍数了
        0
    }
}

struct.alignment = struct.fields().map(|field| field.alignment).max();

let current_offset = 0;

for field in struct.fields_in_declaration_order() {
    // 增加当前字的偏移量段(`current_offset`)，使其成为该字段对齐量的倍数。
    // 对于第一个字段，此值始终为零。
    // 跳过的字节称为填充字节。
    current_offset += padding_needed_for(current_offset, field.alignment);

    struct[field].offset = current_offset;

    current_offset += field.size;
}

struct.size = current_offset + padding_needed_for(current_offset, struct.alignment);
```

<div class="warning">

警告:这个伪代码使用了一个简单粗暴的算法，是为了清晰起见，它忽略了溢出问题。要在实际代码中执行内存布局计算，请使用 [`Layout`]。

</div>

> 注意：此算法可以生成零尺寸的结构体。在C语言中，像 `struct Foo { }` 这样的空结构体声明是非法的。然而，gcc 和 clang 都支持启用此类结构体的选项，并将其尺寸指定为零。跟 Rust 不同的是 C++ 给空结构体指定的尺寸为1，并且除非它们是继承的，否则它们是具有 `[[no_unique_address]]` 属性的字段(在这种情况下，它们不会增大结构体的整体尺寸)。

#### \#[repr(C)] Unions
#### \#[repr(C)] 联合体

使用 `#[repr(C)]` 声明的联合体将与相同目标平台上的C语言中的 C联合体声明具有相同的尺寸和对齐量。联合体的对齐量等同于其所有字段的最大对齐量，尺寸将为其所有字段的最大对尺寸，再对其向上取整到最近对齐量的整数倍。这些最大值可能来自不同的字段。

```rust
#[repr(C)]
union Union {
    f1: u16,
    f2: [u8; 4],
}

assert_eq!(std::mem::size_of::<Union>(), 4);  // 来自于 f2
assert_eq!(std::mem::align_of::<Union>(), 2); // 来自于 f1

#[repr(C)]
union SizeRoundedUp {
   a: u32,
   b: [u16; 5],
}

assert_eq!(std::mem::align_of::<SizeRoundedUp>(), 4); // 来自于 a

assert_eq!(std::mem::size_of::<SizeRoundedUp>(), 12);  // 首先来自于b的尺寸10，然后向上取整到最近的4的整数倍12。

```

#### \#[repr(C)] Field-less Enums
#### \#[repr(C)] 无字段枚举

对于[无字段枚举][field-less enums]，C表形的尺寸和对齐量与目标平台的 C ABI 的默认枚举尺寸和对齐量相同。

> 注意：C中的枚举的表示形式是由实现定义的，所以实际应用中这很可能是一个“最佳猜测”。特别是，当使用某些标志参数来编译感兴趣的C代码时，这可能是不正确的。<!-- Note: The enum representation in C is implementation defined, so this is really a "best guess". In particular, this may be incorrect when the C code of interest is compiled with certain flags. TobeModi-->

<div class="warning">

警告：C语言中的枚举与 Rust 的[无字段枚举][field-less enums]之间有着重要的区别。C语言中的枚举主要是 `typedef` 加上一些命名常量；换句话说，枚举类型的对象可以包含任何整数值。例如，它通常用于C语言中的位标志。相比之下，Rust的[无字段枚举][field-less enums]只能合法地保存判别式的值，其他的都是[未定义行为][undefined behavior]。因此，在FFI中使用无字段枚举来建模 C语言中的枚举（`enum`）通常是错误的。

</div>

#### \#[repr(C)] Enums With Fields
#### \#[repr(C)] 带字段枚举

带字段的 `repr(C)`枚举的表形相当于一个带两个字段的 `repr(C)`结构体（这种在C语言中也被称为“标签联合(tagged union)”），这两个字段：

- 一个为 `repr(C)` 版本的枚举（在这个结构体内，它也被叫做“the tag”），它就是原枚举的判别式组成的新类型，也就是它的变体是原枚举变体移除了它们自身所有的字段。
- 一个为 `repr(C)` 版本的联合体（在这个结构体内，它也被叫做“the payload”），它的各个字段就是原枚举的各个变体的字段组成的 `repr(C)` 版本的结构体。

> 注意：由于是 `repr(C)` 版本的结构体和联合体，如果某变体只单个字段，则直接将该字段放入联合体或将其包装进结构体中没有区别；因此，任何希望操终此类枚举表现形式的系统都可以使用对它们自己来说更方便或更一致的形式。

```rust
// 这个枚举的表形等效于 ...
#[repr(C)]
enum MyEnum {
    A(u32),
    B(f32, u64),
    C { x: u32, y: u8 },
    D,
 }

// ... 这个结构体
#[repr(C)]
struct MyEnumRepr {
    tag: MyEnumDiscriminant,
    payload: MyEnumFields,
}

// 这是原判别式组成的新枚举类型.
#[repr(C)]
enum MyEnumDiscriminant { A, B, C, D }

// 这是原变体的字段组成的联合体.
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

// 这个结构体可以被省略(它是一个零尺寸类型)，但它必须在 C/C++ 头文件中
#[repr(C)]
#[derive(Copy, Clone)]
struct MyDFields;
```

> 注意： 联合体(`union`)带有未实现 `Copy` trait 的字段的功能还没有纳入稳定版，参见[55149]。

### Primitive representations
### 原语表形

*原语表形*是与原生整型具有相同名称的表形。也就是：`u8`，`u16`，`u32`，`u64`，`u128`，`usize`，`i8`，`i16`，`i32`，`i64`，`i128`，和 `isize`。

#### Primitive Representation of Field-less Enums
#### 无字段枚举的原语表形

对于[无字段枚举][field-less enums]，原语表形将其尺寸和对齐量设置成与给定表形同名的原生类型的表形的值。例如，一个 `u8`表形的无字段枚举只能有0和255之间的判别值。

#### Primitive Representation of Enums With Fields
#### 带字段枚举的原语表形

枚举的原语表形是一个 `repr(C)`版本的联合体。此联合体的每个字段对应一个和原枚举变体对应的 `repr(C)`版本的结构体。这些结构体的第一个字段是原枚举的变体移除了它们所有的字段组成的原语表形版本的枚举（“the tag”），那这些结构体的其余字段是原变体移走的字段。

> 注意：如果标签(“the tag”)在联合体中被赋予自己的成员，那么这种表型是不变的，这样操作对您来说会更清晰(尽管遵循c++标准，标签成员应该被包装在结构体中）。
<!-- Note: This representation is unchanged if the tag is given its own member in the union, should that make manipulation more clear for you (although to follow the C++ standard the tag member should be wrapped in a `struct`). -->

```rust
// 这个枚举的表形效同于 ...
#[repr(u8)]
enum MyEnum {
    A(u32),
    B(f32, u64),
    C { x: u32, y: u8 },
    D,
 }

// ... 这个联合体.
#[repr(C)]
union MyEnumRepr {
    A: MyVariantA,
    B: MyVariantB,
    C: MyVariantC,
    D: MyVariantD,
}

// 这是那个判别值组成的枚举。
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
#### 带字段枚举的原语表形与 \#[repr(C)] 表形的组合使用

对于带字段枚举，还可以将 `repr(C)` 和原语表形(例如，`repr(C, u8)`)结合起来使用。这是通过将判别值组成枚举的表形改为原语表形来实现的。因此，如果选择 `u8`表形，那么判别值枚举的尺寸和对齐量将为1个字节。

那么这个判别值枚举就[前面][`repr(C)`]示例中的样子变成：

```rust
#[repr(C, u8)] // 这里加上了 `u8`
enum MyEnum {
    A(u32),
    B(f32, u64),
    C { x: u32, y: u8 },
    D,
 }

// ...

#[repr(u8)] // 所以这里就用 `u8` 替代了 `C`
enum MyEnumDiscriminant { A, B, C, D }

// ...
```

例如，对于有 `repr(C, u8)`属性的枚举，不可能有257个唯一的判别值（“tags”），而同一个只有 `repr(C)`属性的枚举编译时就不会出现任何问题。

在 `repr(C)` 附加原语表形可以改变 `repr(C)`表形的枚举的尺寸：
Using a primitive representation in addition to `repr(C)` can change the size of an enum from the `repr(C)` form:

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

// C表形的尺寸依赖于平台
assert_eq!(std::mem::size_of::<EnumC>(), 8);
// 一个字节用于判别值，一个字节用于 Enum8::Variant0 中的值
assert_eq!(std::mem::size_of::<Enum8>(), 2);
// 两个字节用于判别值，一个字节用于Enum16::Variant0中的值，加上一个字节的填充
assert_eq!(std::mem::size_of::<Enum16>(), 4);
```

[`repr(C)`]: #reprc-enums-with-fields

### The alignment modifiers
### 对齐量的修饰符

`align` 和 `packed` 修饰符可分别用于增大或缩小结构体和联合体的对齐量。`packed` 也可以改变字段之间的填充。

对齐量被指定为整型参数，形式为 `#[repr(align(x))]` 或 `#[repr(packed(x))]`。对齐量的值必须是从1到2<sup>29</sup>之间的2的次幂数。对于packed，如果没有给出任何值，如#[repr(packed)]，则值为1
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
