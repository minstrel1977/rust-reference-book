## Memory ownership
## 内存所有权

>[memory-ownership.md](https://github.com/rust-lang/reference/blob/master/src/memory-ownership.md)\
>commit: 4a2bdf896cd2df370a91d14cb8ba04e326cd21db

当退出堆栈帧时，它的本地分配将全部释放，并且它持有的 box引用将被删除。