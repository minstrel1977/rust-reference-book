# Application Binary Interface (ABI)
# 应用程序二进制接口(ABI)

>[abi.md.md](https://github.com/rust-lang/reference/blob/master/src/abi.md)\
>commit:  24137b49a3c02f53e5a7699a78a47501aef3e769

本节阐述影响 crate 编译输出的ABI的特性。

有关指定用于导出函数(exporting functions)的ABI的信息，请参阅[*外部函数*][extern functions]。有关指定用于链接外部库的ABI的信息，请参阅[*外部块*][external blocks]。

## The `used` attribute
## `used`属性

*`used`属性*只能应用于[静态(`static`)项][`static` items]。此[属性][attribute]强制编译器将该变量保留在输出对象文件中(.o，.rlib等，不包括最终二进制文件)，即便该变量没有被 crate 中的任何其他项使用或引用。但是链接器(linker)仍有权移除此类变量。

下面的示例显示了编译器在什么条件下在输出对象文件中保留静态项。

``` rust
// foo.rs

// 将保留，因为 `#[used]`:
#[used]
static FOO: u32 = 0;

// 可移除，因为没实际使用:
#[allow(dead_code)]
static BAR: u32 = 0;

// 将保留，因为这个是公有的:
pub static BAZ: u32 = 0;

// 将保留，因为这个被可达公有函数引用:
static QUUX: u32 = 0;

pub fn quux() -> &'static u32 {
    &QUUX
}

// 可移除，因为被私有，未被使用的函数引用:
static CORGE: u32 = 0;

#[allow(dead_code)]
fn corge() -> &'static u32 {
    &CORGE
}
```

``` console
$ rustc -O --emit=obj --crate-type=rlib foo.rs

$ nm -C foo.o
0000000000000000 R foo::BAZ
0000000000000000 r foo::FOO
0000000000000000 R foo::QUUX
0000000000000000 T foo::quux
```

## The `no_mangle` attribute
## `no_mangle`属性

可以在任何[数据项][item]上使用 *`no_mangle`属性*来禁用标准符号名称混淆(standard symbol name mangling)。数据项的符号是数据项的名称的标识符。

## The `link_section` attribute
## `link_section`属性

`link_section`属性指定了对象文件中[函数][function]或[静态项][static]的内容将被放置到的节点位置。它使用 [_MetaNameValueStr_]元项属性句法规则指定节点名称。

<!-- no_run: don't link. The format of the section name is platform-specific. -->
```rust,no_run
#[no_mangle]
#[link_section = ".example_section"]
pub static VAR1: u32 = 1;
```

## The `export_name` attribute
## `export_name`属性

*`export_name`属性*指定将在[函数][function]或[静态项][static]上导出的符号的名称。它使用 [_MetaNameValueStr_]元项属性句法规则指定符号名。

```rust
#[export_name = "exported_symbol_name"]
pub fn name_in_rust() { }
```

[_MetaNameValueStr_]: attributes.md#元项属性句法
[`static` items]: items/static-items.md
[attribute]: attributes.md
[extern functions]: items/functions.md#外部函数限定符
[external blocks]: items/external-blocks.md
[function]: items/functions.md
[item]: items.md
[static]: items/static-items.md
