# Memory allocation and lifetime
# 内存分配和生存期

>[memory-allocation-and-lifetime.md](https://github.com/rust-lang/reference/blob/master/src/memory-allocation-and-lifetime.md)\
>commit af1cf6d3ca3b7a8c434c142148742aa912e37c34

程序的*数据项*是那些函数、模块和类型，它们的值在编译时被计算出来，并且唯一地存储在 Rust 进程的内存映像中。数据项既不是动态分配的，也不是动态释放的。

*堆*是描述 box封装类型数据（译者注：封装是一个堆分配过程，该封装或者说堆分配返回一个指向该堆分配的内存地址的引用，后文称这个引用为 box值或 box引用）的通用术语。堆分配的生存期取决于指向它的 box值的生存期。由于 box值本身可能被传入或传出栈帧，或者存储在堆中，因此堆分配可能比初始分配它们的栈帧存活的时间长。在堆分配的整个生存期内，该堆分配被保证驻留在堆中的单个位置 — 它永远不会因移动 box值而重新分配内存地址。
