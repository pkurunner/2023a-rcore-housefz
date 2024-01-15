**ch5报告**

**①迁移get_time，get_task_info，mmap和munmap**：



get_time：与之前相同。

get_task_info，mmap和munmap：用current_task得当前进程。其余步骤与之前相同，都实现为TaskControlBlock的方法。



②**实现系统调用spawn**



将fork和exec连在一起，与fork的不同在于，子进程的TaskControlBlock的参数，改为path获取的elf_data中的参数（不复制父进程的地址空间）。



③**实现stride调度算法**。



在TaskControlBlockInner添加priority和stride两个变量，sys_set_priority将当前进程的优先级修改为对应的参数即可。调度算法修改TaskManager的fetch函数，遍历等待队列，获得stride最小的进程，根据stride += BIG_STRIDE / priority，再从队列中弹出这个进程。BIG_STRIDE取720720。