## Memory ownership
## 内存所有权

>[memory-ownership.md](https://github.com/rust-lang/reference/blob/master/src/memory-ownership.md)\
>commit: 4a2bdf896cd2df370a91d14cb8ba04e326cd21db \
>本章译文最后维护日期：2020-11-2

当离开堆栈帧时，它的本地分配将全部释放，并且它持有的 box指针将被删除。

<!-- 2020-11-7-->
<!-- checked -->
