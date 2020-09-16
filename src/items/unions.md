# 联合体

>[unions.md](https://github.com/rust-lang/reference/blob/master/src/items/unions.md)\
>commit ad8986882c6b069cfbdca6103746f5a3e8cb9d5b

> **<sup>句法</sup>**\
> _Union_ :\
> &nbsp;&nbsp; `union` [IDENTIFIER]&nbsp;[_Generics_]<sup>?</sup> [_WhereClause_]<sup>?</sup>
>   `{`[_StructFields_] `}`

联合体声明使用和结构体声明相同的句法格式，除了用 `union` 代替 `struct`。

```rust
#[repr(C)]
union MyUnion {
    f1: u32,
    f2: f32,
}
```

联合体的关键属性是联合体的所有字段共享同一段公用存储。因此，对联合体的一个字段的写操作会覆盖其他字段，而联合体的尺寸由其尺寸最大的字段的尺寸决定。

## 联合体的初始化

可以使用与结构体类型相同的语法创建联合体类型的值，但必须只能指定一个字段：

```rust
# union MyUnion { f1: u32, f2: f32 }
#
let u = MyUnion { f1: 1 };
```

上面的表达式创建类型为 `MyUnion` 的值，并使用字段 `f1` 初始化存储。可以使用与结构体字段相同的语法访问联合体：

```rust
# union MyUnion { f1: u32, f2: f32 }
#
# let u = MyUnion { f1: 1 };
let f = unsafe { u.f1 };
```

## 读写联合体字段

联合体没有“活跃字段”的概念。相反，每此访问联合体只用访问的字段类型解释的此联合体的数据存储。读取联合体字段将以当前读取字段的类型来解读此联合体的存储比特。字段（间）可以有非零的偏移量存在（使用 `#[repr(C)]` 的除外）；在这种情况下读取，从字段偏移量开始的比特位将被读取。程序员有责任确保数据在当前字段类型下有效。否则会导致未定义的行为。例如，在 `bool` 类型下读取到数值 `3` 是未定义的行为。实际上，对一个 `#[repr(C)]` 联合体进行写操作，然后再从中读取，就好比从用于写入的类型到用于读取的类型的 [`transmute`] 操作。

因此，所有联合体字段的读取必须放在 `unsafe` 块里：

```rust
# union MyUnion { f1: u32, f2: f32 }
# let u = MyUnion { f1: 1 };
#
unsafe {
    let f = u.f1;
}
```

因为对拥有 `Copy` 特性的联合体字段的写操作不需要事先的析构读操作[^译者备注]，所以这些写操作不必放在 `unsafe` 块中。

```rust
# union MyUnion { f1: u32, f2: f32 }
# let mut u = MyUnion { f1: 1 };
#
u.f1 = 2;
```

通常，使用联合体的代码将为非安全的联合体字段访问提供一层安全包装。

## 联合体上的模式匹配

访问联合体字段的另一种方法是使用模式匹配。联合体字段上的模式匹配与结构体上的模式匹配使用相同的语法，只是这种模式只能指定一个字段。因为模式匹配就像使用特定字段来读取联合体，所以它也必须被放在 `unsafe` 块中。

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

模式匹配可以将联合体作为更大的数据结构的字段进行匹配。特别是，当使用 Rust 联合体通过 FFI 实现 C 标记的联合体时，这允许同时在标签和相应字段上进行匹配:

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

## 引用联合体字段

由于联合体字段共享公共存储，因此拥有对联合体一个字段的写访问权就同时拥有了对为其所有其他字段的写访问权。因为这一事实，引用的借用检查规则必须调整。因此，如果联合体的一个字段是被出借，那么它的所有其他字段在相同的生命周期内也都处于出借状态。

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

正如您所看到的，在许多方面(除了布局、安全性和所有权)，联合体的行为与结构体完全相同，这很大程度上是因为联合体继承了结构体的句法格式的结果。对于 Rust 语言未明确提及的许多方面(比如隐私性、名称解析、类型推断、泛型、trait 实现、固有实现、一致性、模式检查等等)也是如此。

[^译者备注]: 这句译者简单理解就是对已经初始化的变量再去覆写的时候要先去读一下这个变量代表的地址上的值的状态，如果有值，并且允许覆写，并且值没有 Copy 特性，那Rust 未防止内存泄漏就先执行那变量的析构行为（drop()），清空那个地址上的数据，再写入。我们这里的预设条件是那值有 Copy 特性，有Copy 特性了，对值的直接覆写不会造成内存泄漏，就不需要事先的非安全的读操作了。对于这个问题[nomicon](https://learnku.com/docs/nomicon/2018)的“未初始化内存”章有讲述，博主[CrLF0710](https://www.zhihu.com/people/crlf0710)的两篇“学一点 Rust 内存模型会发生什么呢？”里也都有精彩讲解。

[IDENTIFIER]: ../identifiers.md
[_Generics_]: generics.md
[_WhereClause_]: generics.md#where-clauses
[_StructFields_]: structs.md
[`transmute`]: ../../std/mem/fn.transmute.html
