# Debugger attributes
# 调试器属性

>[debugger.md](https://github.com/rust-lang/reference/blob/master/src/attributes/debugger.md)\
>commit: 39a0be6dc8210e0f127638ac5cc6820b8ce534df \
>本章译文最后维护日期：2023-06-11

以下[属性][attributes]用于在使用 GDB 或 WinDbg 等第三方调试器时增强调试体验。

## The `debugger_visualizer` attribute
## `debugger_visualizer`属性

*`debugger_visualizer`属性*可用于将可视化的调试器工具文件嵌入到调试信息中。
这样可以改善调试器在调试过程中显示值时的体验。
它使用[_MetaListNameValueStr_]句法来指定其输入，并且必须指定为 crate属性。

### Using `debugger_visualizer` with Natvis
### 将 `debugger_visualizer` 与 Natvis 一起使用

Natvis 是一个用于 Microsoft调试器（如 Visual Studio 和 WinDbg）的基于 XML 的框架，它使用声明性规则来自定义类型的显示。
有关 Natvis格式的详细信息，请参阅 Microsoft 的[Natvis文档][Natvis documentation]。

此属性仅支持在 `-windows-msvc`目标平台上嵌入 Natvis文件。

Natvis文件的路径由 `natvis_file`键指定，该键是相对于 crate源文件的路径：

<!-- ignore: requires external files, and msvc -->
```rust ignore
#![debugger_visualizer(natvis_file = "Rectangle.natvis")]

struct FancyRect {
    x: f32,
    y: f32,
    dx: f32,
    dy: f32,
}

fn main() {
    let fancy_rect = FancyRect { x: 10.0, y: 10.0, dx: 5.0, dy: 5.0 };
    println!("set breakpoint here");
}
```

其中 `Rectangle.natvis` 包含:

```xml
<?xml version="1.0" encoding="utf-8"?>
<AutoVisualizer xmlns="http://schemas.microsoft.com/vstudio/debugger/natvis/2010">
    <Type Name="foo::FancyRect">
      <DisplayString>({x},{y}) + ({dx}, {dy})</DisplayString>
      <Expand>
        <Synthetic Name="LowerLeft">
          <DisplayString>({x}, {y})</DisplayString>
        </Synthetic>
        <Synthetic Name="UpperLeft">
          <DisplayString>({x}, {y + dy})</DisplayString>
        </Synthetic>
        <Synthetic Name="UpperRight">
          <DisplayString>({x + dx}, {y + dy})</DisplayString>
        </Synthetic>
        <Synthetic Name="LowerRight">
          <DisplayString>({x + dx}, {y})</DisplayString>
        </Synthetic>
      </Expand>
    </Type>
</AutoVisualizer>
```

当在 WinDbg 下查看时， `fancy_rect`变量将显示如下信息：

```text
> Variables:
  > fancy_rect: (10.0, 10.0) + (5.0, 5.0)
    > LowerLeft: (10.0, 10.0)
    > UpperLeft: (10.0, 15.0)
    > UpperRight: (15.0, 15.0)
    > LowerRight: (15.0, 10.0)
```

### Using `debugger_visualizer` with GDB
### `debugger_visualizer` 与 GDB 一起使用

GDB 支持使用一个结构化的 Python脚本，称为 *靓化打印输出（pretty printer）*，该脚本描述了如何在调试器视图中打印输出可视化类型。
有关靓化打印输出的详细信息，请参阅GDB的[靓化打印输出文档][pretty printing documentation]。

在 GDB 下调试二进制文件时，嵌入式的靓化打印输出不会被自动加载。
有两种方法可以启用自动加载的嵌入的式靓化打印输出：
1.使用额外的参数启动 GDB，将文件夹或二进制文件显式地添加到自动加载安全路径：`GDB-iex“add auto-load safe path path path/to/binary”path/to/binary`
有关更多信息，请参阅GDB的[自动加载文档]。
1.在 `$HOME/.config/gdb` 下创建一个名为 `gdbinit` 的文件（如果该文件夹还不存在，则可能需要创建该文件夹）。在该文件中添加以下行：`add-auto-load-safe-path path/to/binary`
。
这些脚本是使用 `gdb_script_file`键嵌入的，该键是相对于 crate源文件的路径。

<!-- ignore: requires external files -->
```rust ignore
#![debugger_visualizer(gdb_script_file = "printer.py")]

struct Person {
    name: String,
    age: i32,
}

fn main() {
    let bob = Person { name: String::from("Bob"), age: 10 };
    println!("set breakpoint here");
}
```

其中 `printer.py` 包含:

```python
import gdb

class PersonPrinter:
    "Print a Person"

    def __init__(self, val):
        self.val = val
        self.name = val["name"]
        self.age = int(val["age"])

    def to_string(self):
        return "{} is {} years old.".format(self.name, self.age)

def lookup(val):
    lookup_tag = val.type.tag
    if lookup_tag is None:
        return None
    if "foo::Person" == lookup_tag:
        return PersonPrinter(val)

    return None

gdb.current_objfile().pretty_printers.append(lookup)
```

当此 crate的可调试的二进制文件被传递到 GDB[^rust-gdb] 时，`print bob` 将显示：

```text
"Bob" is 10 years old.
```

[^rust-gdb]: 注意：这里假设你使用的是 `rust-gdb`脚本，该脚本使用了 `String`之类的标准库里的类型来配置靓化打印输出。

[auto-loading documentation]: https://sourceware.org/gdb/onlinedocs/gdb/Auto_002dloading-safe-path.html
[attributes]: ../attributes.md
[Natvis documentation]: https://docs.microsoft.com/en-us/visualstudio/debugger/create-custom-views-of-native-objects
[pretty printing documentation]: https://sourceware.org/gdb/onlinedocs/gdb/Pretty-Printing.html
[_MetaListNameValueStr_]: ../attributes.md#meta-item-attribute-syntax
