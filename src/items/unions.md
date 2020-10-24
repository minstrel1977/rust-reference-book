# Unions
# 联合体

>[unions.md](https://github.com/rust-lang/reference/blob/master/src/items/unions.md)\
>commit: ad8986882c6b069cfbdca6103746f5a3e8cb9d5b \
>本译文最后维护日期：2020-10-21

> **<sup>句法</sup>**\
> _Union_ :\
> &nbsp;&nbsp; `union` [IDENTIFIER]&nbsp;[_Generics_]<sup>?</sup> [_WhereClause_]<sup>?</sup>
>   `{`[_StructFields_] `}`

联合体声明使用和结构体声明相同的句法规则，除了用 `union` 代替 `struct`。

```rust
#[repr(C)]
union MyUnion {
    f1: u32,
    f2: f32,
}
```

联合体的关键属性是联合体的所有字段共享同一段存储。因此，对联合体的一个字段的写操作会覆盖其他字段，而联合体的尺寸由其尺寸最大的字段的尺寸决定。

## Initialization of a union
## 联合体的初始化

可以使用与结构体类型相同的句法创建联合体类型的值，但必须只能指定一个字段：

```rust
# union MyUnion { f1: u32, f2: f32 }
#
let u = MyUnion { f1: 1 };
```

上面的表达式创建了一个类型为 `MyUnion` 的值，并使用字段 `f1` 初始化了其存储。可以使用与结构体字段相同的句法访问联合体：

```rust
# union MyUnion { f1: u32, f2: f32 }
#
# let u = MyUnion { f1: 1 };
let f = unsafe { u.f1 };
```

## Reading and writing union fields
## 读写联合体字段

联合体没有“活跃字段(active field)”的概念。相反，每次访问联合体只用访问所指定的字段的类型解释此联合体的数据存储。读取联合体字段将以当前读取字段的类型来解读此联合体的存储位。字段（间）可以有非零的偏移量存在（使用表型 `#[repr(C)]` 的除外）；在这种情况下，读取将从字段的相对偏移量的比特位开始。程序员有责任确保数据在当前字段类型下有效。否则会导致未定义行为(undefined behavior)。例如，在 `bool` 类型下读取到数值 `3` 是未定义行为。实际上，对一个 `#[repr(C)]` 表型的联合体进行写操作，然后再从中读取，就好比从用于写入的类型到用于读取的类型的 [`transmute`] 操作。

因此，所有的联合体字段的读取必须放在非安全(`unsafe`)块里：

```rust
# union MyUnion { f1: u32, f2: f32 }
# let u = MyUnion { f1: 1 };
#
unsafe {
    let f = u.f1;
}
```

因为对拥有 `Copy` 特性的联合体字段的写操作不需要事先的析构读操作[^译者备注]，所以这些写操作不必放在非安全(`unsafe`)块中。

```rust
# union MyUnion { f1: u32, f2: f32 }
# let mut u = MyUnion { f1: 1 };
#
u.f1 = 2;
```

通常，使用联合体的代码将为非安全的联合体字段访问提供一层安全包装。

## Pattern matching on unions
## 联合体上的模式匹配

访问联合体字段的另一种方法是使用模式匹配。联合体字段上的模式匹配与结构体上的模式匹配使用相同的句法规则，只是这种模式只能一次指定一个字段。因为模式匹配就像使用特定字段来读取联合体，所以它也必须被放在非安全(`unsafe`)块中。

```rust
# union MyUnion { f1: u32, f2: f32 }
#
fn f(u: MyUnion) {
    unsafe {
        match u {
            MyUnion { f1: 10 } => { println!("ten"); }
            MyUnion { f2 } => { println!("{}", f2); }
        }
    }
}
```

模式匹配可以将联合体作为更大的数据结构的一个字段进行匹配。特别是，当使用 Rust 联合体通过 FFI 实现 C标签联合体(C tagged union)时，这允许同时在标签和相应字段上进行匹配：

```rust
#[repr(u32)]
enum Tag { I, F }

#[repr(C)]
union U {
    i: i32,
    f: f32,
}

#[repr(C)]
struct Value {
    tag: Tag,
    u: U,
}

fn is_zero(v: Value) -> bool {
    unsafe {
        match v {
            Value { tag: Tag::I, u: U { i: 0 } } => true,
            Value { tag: Tag::F, u: U { f: num } } if num == 0.0 => true,
            _ => false,
        }
    }
}
```

## References to union fields
## 引用联合体字段

Since union fields share common storage, gaining write access to one field of a
union can give write access to all its remaining fields. Borrow checking rules
have to be adjusted to account for this fact. As a result, if one field of a
union is borrowed, all its remaining fields are borrowed as well for the same
lifetime.
由于联合体字段共享存储，因此拥有对联合体一个字段的写访问权就同时拥有了对其所有其他字段的写访问权。因为这一事实，引用的借用检查规则必须调整。因此，如果联合体的一个字段是被出借，那么在相同的生存期内它的所有其他字段也都处于出借状态。

```rust,compile_fail
# union MyUnion { f1: u32, f2: f32 }
// 错误: 不能同时对 `u` (通过 `u.f2`)拥有多余一次的可变借用
fn test() {
    let mut u = MyUnion { f1: 1 };
    unsafe {
        let b1 = &mut u.f1;
//                    ---- 首次可变借用发生在这里 (通过 `u.f1`)
        let b2 = &mut u.f2;
//                    ^^^^ 二次可变借用发生在这里 (通过 `u.f2`)
        *b1 = 5;
    }
//  - 首次借用在这里结束
    assert_eq!(unsafe { u.f1 }, 5);
}
```

正如您所看到的，在许多方面(除了布局、安全性和所有权)，联合体的行为与结构体完全相同，这很大程度上是因为联合体继承了结构体的句法规则的结果。对于 Rust 语言未明确提及的许多方面(比如隐私性、名称解析、类型推断、泛型、trait 实现、固有实现、一致性、模式检查等等)也是如此。

[^译者备注]: 这句译者简单理解就是对已经初始化的变量再去覆写的时候要先去读一下这个变量代表的地址上的值的状态，如果有值，并且允许覆写，那Rust 为防止内存泄漏就先执行那变量的析构行为（drop()），清空那个地址上的数据的关联数据，再写入。我们这里的预设条件是那值有 Copy 特性，有Copy 特性了，对值的直接覆写不会造成内存泄漏，就不必调用析构行为，也不需要事先的非安全读操作了。对于这个问题[nomicon](https://learnku.com/docs/nomicon/2018)的“未初始化内存”章有讲述，博主[CrLF0710](https://www.zhihu.com/people/crlf0710)的两篇“学一点 Rust 内存模型会发生什么呢？”里也都有精彩讲解。

[IDENTIFIER]: ../identifiers.md
[_Generics_]: generics.md
[_WhereClause_]: generics.md#where-clauses
[_StructFields_]: structs.md
[`transmute`]: https://doc.rust-lang.org/std/mem/fn.transmute.html

<!-- 2020-10-16 -->
<!-- checked -->
