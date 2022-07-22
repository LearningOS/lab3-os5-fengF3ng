# 实现的功能

首先在task中实现了spawn接口将输入的elf数据打包成新的task_control_block，在sys_spawn中通过path读取elf_data并按照之前的接口生成task_control_block，将其add进task队列即可。

在stride算法的实现中，考虑到每次进程yield主要逻辑在suspend_current_and_run_next函数中，便在此增加进程的pass长度。对于每次调度，在manager中使用二叉堆，以pass为关键值进行排序，总是pop一个pass最小的进程。

mmap和munmap实现逻辑与之前相同，不过由于这次memory_set提供了全新的接口，所以更简易的实现了分配和撤销功能。

# 1

不会，p2的pass溢出了，变成了数值4成为pass最小的进程。

stride<=BigStride/2，假设存在PASS_MAX-PASS_MIN>BigStride/2的情况，那么经过几轮调用后总会达到PASS_MAX-PASS_MIN=BigStride/2 + A，其中0< A <=stride的状态，此时再减去一个stride就有PASS_MAX-PASS_MIN <= BigStride/2

```Rust
use core::cmp::Ordering;

struct Pass(u64);

impl PartialOrd for Pass {
    fn partial_cmp(&self, other: &Self) -> Option<Ordering> {
        if self.pass - other.pass > BigStride / 2 {
            other.pass.cmp(&self.pass)
        } else {
            self.pass.cmp(&other.pass)
        }
    }
}

impl PartialEq for Pass {
    fn eq(&self, other: &Self) -> bool {
        false
    }
}
```