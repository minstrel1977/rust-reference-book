# Linkage
# 联接

>[linkage.md](https://github.com/rust-lang/reference/blob/master/src/linkage.md)\
>commit 79364a6583803c270ff5be1085434631aba30858

> 注意：本节更多的是从编译器的角度来描述的，而不是语言。

编译器支持多种将板条箱静态地和动态地联接在一起的方法。本节将探索各种将 crate 联接在一起的方法，关于本地库的更多信息请参阅[本书的FFI部分][ffi]。

[ffi]: ../book/ffi.html

在一个编译会话中，编译器可以通过使用命令行参数或 `crate_type` 属性来生成多个构件(artifacts)。如果指定了一个或多个命令行参数，则将忽略 carte 内部所有的 `crate_type` 属性，以便只构建由命令行指定的构件。

* `--crate-type=bin`, `#[crate_type = "bin"]` - 将生成一个可执行文件。这就要求在 crate 中有一个 `main` 函数，它将在程序开始执行时运行。这将联接所有 Rust 和本地依赖文件，生成一个可分发的二进制文件。

* `--crate-type=lib`, `#[crate_type = "lib"]` - 将生成一个 Rust 库文件(library)。对于这么指定，底层到底生成了什么，这是一个模糊的概念，因为库文件有多种表现形式。使用常用的 `lib` 选项的目的是生成“编译器推荐”类型的库文件。像这里这种指定输出库文件类型的选项在 rustc 始终可用，但是每次实际输出的库文件的类型可能会随实际情况而不同。其它的输出类型指定方式都指向具体类型的库文件，而 `lib` 可以看作是那些类型中的某个类型的别名(具体实际的输出类型是编译器决定的)。

* `--crate-type=dylib`, `#[crate_type = "dylib"]` - 将生成一个动态 Rust 库文件。这与 `lib` 输出类型不同，因为它强制生成动态库文件。生成的动态库文件可以用作其他库文件和/或可执行文件的依赖文件。这个输出类型将创建依赖于具体平台的库文件（linux 上为 `*.so`，osx 上为 `*.dylib`、windows 上为 `*.dll`）。

* `--crate-type=staticlib`, `#[crate_type = "staticlib"]` - 将生成一个静态系统库文件。这个与其他类型的库文件输出的不同之处在于，编译器永远不会尝试去联接 `staticlib` 输出。此输出类型的目的是创建一个包含所有本地 crate 的代码以及所有上游依赖文件的静态库文件。静态库文件实际上是 linux、osx 和 windows(MinGW) 平台上的 `*.a` 归档文件(archive)，或者 windows(MSVC) 平台上的 `*.lib` 文件。在这些情况下，例如将 Rust 代码链接到现有的非 Rust 应用程序中，推荐使用这种格式，因为它不会动态依赖于其他 Rust 代码。

* `--crate-type=cdylib`, `#[crate_type = "cdylib"]` - 将生成一个动态系统库文件。如果编译输出的动态库文件要被另一种语言加载使用，请指定这种编译方式。这种输出类型将在 Linux 上创建 `*.so` 文件，在 macOS 上创建 `*.dylib` 文件，在 Windows 上创建 `*.dll` 文件。

* `--crate-type=rlib`, `#[crate_type = "rlib"]` - 将生成一个“Rust库文件”。它被用作一个中间构件，可以被认为是一个“静态 Rust 库文件”。与 `staticlib` 文件不同，这些 `rlib` 文件由未来现场的编译器在现场的联接中解释。这本质上意味着（那时的） `rustc` 将在（此） `rlib` 文件中查找元数据(metadata)，就像在动态库文件中查找元数据一样。跟 `staticlib` 输出类型类似，这种形式的输出常用于生成静态联接的可执行文件(statically linked executable)。

* `--crate-type=proc-macro`, `#[crate_type = "proc-macro"]` - 生成的输出没有被指定，但是如果通过 `-L` 提供了路径参数，编译器将把输出构件识别为宏，输出的宏可以被其他 Rust 程序加载使用。使用此 crate 类型编译的 crate 只能导出[过程宏][procedural macros]。编译器将自动设置 `proc_macro`[属性配置选项][configuration option]。编译 crate 的目标平台总是和当前编译器所在平台一致。例如，如果您在 `x86_64` CPU 的 Linux 平台上执行编译，那么目标平台将是 `x86_64-unknown-linux-gnu`，即使该 crate 是另一个不同构建目标的 crate 的依赖文件。

请注意，这些输出指定是可堆叠使用的，如果指定了多个输出，那么编译器将一次生成所有这些指定形式的输出，而不必反复多次编译。但是，这只适用于由相同方法指定的输出。如果只指定了 `crate_type` 属性，则将生成所有类型的输出，但如果指定了一个或多个 `--crate-type` 命令行参数，则只生成这些指定的输出。

对于所有这些不同类型的输出，如果 crate A 依赖于 crate B，那么编译器很可能在整个系统中找到多种不同形式的 B。但是，编译器寻找的只有 `rlib` 格式和动态库格式。有了依赖库文件的这两个选项，编译器在某些时候必须在这两种格式之间做出选择。考虑到这一点，编译器在决定使用哪种依赖关系格式时将遵循这些规则：
有了所有这些不同类型的输出，如果机箱A依赖于机箱B，那么编译器可以在整个系统中找到各种不同形式的B。不过，编译器只查找 `rlib` 格式和动态库格式。对于依赖库有这两个选项，编译器必须在这两种格式之间做出选择。考虑到这一点，编译器在确定将使用的依赖项格式时遵循以下规则：
With all these different kinds of outputs, if crate A depends on crate B, then the compiler could find B in various different forms throughout the system. The only forms looked for by the compiler, however, are the `rlib` format and the dynamic library format. With these two options for a dependent library, the compiler must at some point make a choice between these two formats. With this in mind, the compiler follows these rules when determining what format of dependencies will be used:

1. 如果要生成静态库文件，则需要所有上游依赖文件都以 `rlib` 格式可用。这个需求源于不能将动态库文件转换为静态格式的原因

   注意，不可能将本地动态依赖文件联接到静态库文件，在这种情况下，将打印有关所有未联接的本地动态依赖文件的警告

2. 如果生成 `rlib` 文件，则对上游依赖文件的可用格式没有任何限制，仅需要求所有这些文件都可以从其中读出元数据

   原因是 `rlib` 文件不包含它们的任何上游依赖文件。但如果所有（上游的） `rlib` 文件都包含 `libstd.rlib` 的副本，那下游文件的编译效率和执行效率将大幅降低。

3. 如果当前生成可执行文件，并且没有指定 `-C prefer-dynamic` 参数，则首先尝试以 `rlib` 格式查找依赖文件。如果某些依赖文件在 rlib 格式文件中不可用，则尝试动态联接文件(见下文)。

4. 如果当前生成动态联接的动态库文件或可执行文件，则编译器将尝试协调从 rlib 或 dylib 格式的文件里获取可用依赖关系，以创建最终产品。

   编译器的主要目标是确保库文件不会在任何构件中出现多次。例如，如果动态库文件B和C都静态地联接到库文件A，那么crate就不能同时联接到B和C，因为A有两个副本。编译器允许混合使用rlib和dylib格式，但必须满足这一限制
   A major goal of the compiler is to ensure that a library never appears more
   than once in any artifact. For example, if dynamic libraries B and C were
   each statically linked to library A, then a crate could not link to B and C
   together because there would be two copies of A. The compiler allows mixing
   the rlib and dylib formats, but this restriction must be satisfied.

   编译器目前没有实现任何方法来提示库文件应该联接到哪种格式。当动态联接时，编译器将尝试最大化动态依赖，同时仍然允许通过rlib联接某些依赖
   The compiler currently implements no method of hinting what format a library
   should be linked with. When dynamically linking, the compiler will attempt to
   maximize dynamic dependencies while still allowing some dependencies to be
   linked in via an rlib.

   对于大多数情况，如果动态联接，建议将所有库文件作为动态库文件使用。对于其他情况，如果编译器无法确定将每个库文件联接到哪种格式，则会发出警告
   For most situations, having all libraries available as a dylib is recommended
   if dynamically linking. For other situations, the compiler will emit a
   warning if it is unable to determine which formats to link each library with.

通常，——crate-type=bin或——crate-type=lib应该足以满足所有的编译需求，只有在需要对crate的输出格式进行更细粒度的控制时，才可以使用其他选项
In general, `--crate-type=bin` or `--crate-type=lib` should be sufficient for
all compilation needs, and the other options are just available if more
fine-grained control is desired over the output format of a crate.

## Static and dynamic C runtimes
## 静态C运行时和动态C运行时

一般来说，标准库文件努力支持目标的静态联接和动态联接的C运行时。例如，x86_64-pc-windows-msvc和x86_64-unknown-linux-musl目标通常都带有运行时，用户可以选择自己喜欢的运行时。编译器中的所有目标都有一个联接到C运行时的默认模式。通常，默认情况下目标是动态联接的，但也存在默认情况下静态的异常，例如：
The standard library in general strives to support both statically linked and
dynamically linked C runtimes for targets as appropriate. For example the
`x86_64-pc-windows-msvc` and `x86_64-unknown-linux-musl` targets typically come
with both runtimes and the user selects which one they'd like. All targets in
the compiler have a default mode of linking to the C runtime. Typically targets
are linked dynamically by default, but there are exceptions which are static by
default such as:

* `arm-unknown-linux-musleabi`
* `arm-unknown-linux-musleabihf`
* `armv7-unknown-linux-musleabihf`
* `i686-unknown-linux-musl`
* `x86_64-unknown-linux-musl`

C运行时的联接被配置为尊重crt静态目标特性。这些目标特性通常通过编译器本身的标志从命令行配置。例如，启用您将执行的静态运行时
The linkage of the C runtime is configured to respect the `crt-static` target
feature. These target features are typically configured from the command line
via flags to the compiler itself. For example to enable a static runtime you
would execute:

```sh
rustc -C target-feature=+crt-static foo.rs
```

而动态联接到C运行时，你将执行
whereas to link dynamically to the C runtime you would execute:

```sh
rustc -C target-feature=-crt-static foo.rs
```

不支持在C运行时的联接之间切换的目标将忽略这个标志。建议检查生成的二进制文件，以确保在编译器成功之后，如您预期的那样联接了它
Targets which do not support switching between linkage of the C runtime will
ignore this flag. It's recommended to inspect the resulting binary to ensure
that it's linked as you would expect after the compiler succeeds.

板条箱还可以了解C运行时是如何联接的。例如，MSVC上的代码需要根据联接的运行时进行不同的编译(例如使用/MT或/MD)。它目前通过cfg属性目标特性选项导出
Crates may also learn about how the C runtime is being linked. Code on MSVC, for
example, needs to be compiled differently (e.g. with `/MT` or `/MD`) depending
on the runtime being linked. This is exported currently through the
[`cfg` attribute `target_feature` option]:

```rust
#[cfg(target_feature = "crt-static")]
fn foo() {
    println!("the C runtime should be statically linked");
}

#[cfg(not(target_feature = "crt-static"))]
fn foo() {
    println!("the C runtime should be dynamically linked");
}
```

还要注意，Cargo构建脚本可以通过环境变量了解此特性。在构建脚本中，您可以通过
Also note that Cargo build scripts can learn about this feature through
[environment variables][cargo]. In a build script you can detect the linkage
via:

```rust
use std::env;

fn main() {
    let linkage = env::var("CARGO_CFG_TARGET_FEATURE").unwrap_or(String::new());

    if linkage.contains("crt-static") {
        println!("the C runtime will be statically linked");
    } else {
        println!("the C runtime will be dynamically linked");
    }
}
```

[cargo]: ../cargo/reference/environment-variables.html#environment-variables-cargo-sets-for-build-scripts

要在本地使用此特性，通常需要使用RUSTFLAGS环境变量通过Cargo为编译器指定标志。例如，要在MSVC上编译静态联接的二进制文件，就需要执行
To use this feature locally, you typically will use the `RUSTFLAGS` environment
variable to specify flags to the compiler through Cargo. For example to compile
a statically linked binary on MSVC you would execute:

```sh
RUSTFLAGS='-C target-feature=+crt-static' cargo build --target x86_64-pc-windows-msvc
```

[`cfg` attribute `target_feature` option]: conditional-compilation.md#target_feature
[configuration option]: conditional-compilation.md
[procedural macros]: procedural-macros.md
