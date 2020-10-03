# Function item types
# 函数型

>[function-item.md](https://github.com/rust-lang/reference/blob/master/src/types/function-item.md)\
>commit b0e0ad6490d6517c19546b1023948986578fc378

当引用函数项、元组结构体的构造函数或枚举变量时会产生一个*函数项类型*的零尺寸的值。类型显式标识函数——它的名字,它的类型参数,及其早期绑定的寿命参数(但不是一生的后期绑定参数,只分配函数被调用时),所以不需要包含一个实际的价值函数指针,和不需要间接调用函数时
When referred to, a function item, or the constructor of a tuple-like struct or
enum variant, yields a zero-sized value of its _function item type_. That type
explicitly identifies the function - its name, its type arguments, and its
early-bound lifetime arguments (but not its late-bound lifetime arguments,
which are only assigned when the function is called) - so the value does not
need to contain an actual function pointer, and no indirection is needed when
the function is called.

没有直接引用函数项类型的语法，但是编译器会在错误消息中显示类似于fn(u32) -> i32 {fn name}的类型
There is no syntax that directly refers to a function item type, but the
compiler will display the type as something like `fn(u32) -> i32 {fn_name}` in
error messages.

因为函数项类型显式地标识函数，所以不同函数的项类型(不同的项，或具有不同泛型的相同项)是不同的，混合使用它们将创建类型错误
Because the function item type explicitly identifies the function, the item
types of different functions - different items, or the same item with different
generics - are distinct, and mixing them will create a type error:

```rust,compile_fail,E0308
fn foo<T>() { }
let x = &mut foo::<i32>;
*x = foo::<u32>; //~ ERROR mismatched types
```

然而,有一个从函数项强制函数指针使用相同的签名,这是引发不仅当函数项时使用函数指针直接预期,而且当不同的函数签名相同的项类型满足不同部门相同的如果或匹配
However, there is a [coercion] from function items to [function pointers] with
the same signature, which is triggered not only when a function item is used
when a function pointer is directly expected, but also when different function
item types with the same signature meet in different arms of the same `if` or
`match`:

```rust
# let want_i32 = false;
# fn foo<T>() { }

// `foo_ptr_1` has function pointer type `fn()` here
let foo_ptr_1: fn() = foo::<i32>;

// ... and so does `foo_ptr_2` - this type-checks.
let foo_ptr_2 = if want_i32 {
    foo::<i32>
} else {
    foo::<u32>
};
```

所有的功能项目实现Fn, FnMut, FnOnce，复制，克隆，发送和同步
All function items implement [`Fn`], [`FnMut`], [`FnOnce`], [`Copy`],
[`Clone`], [`Send`], and [`Sync`].

[`Clone`]: ../special-types-and-traits.md#clone
[`Copy`]: ../special-types-and-traits.md#copy
[`FnMut`]: https://doc.rust-lang.org/std/ops/trait.FnMut.html
[`FnOnce`]: https://doc.rust-lang.org/std/ops/trait.FnOnce.html
[`Fn`]: https://doc.rust-lang.org/std/ops/trait.Fn.html
[`Send`]: ../special-types-and-traits.md#send
[`Sync`]: ../special-types-and-traits.md#sync
[coercion]: ../type-coercions.md
[function pointers]: function-pointer.md
