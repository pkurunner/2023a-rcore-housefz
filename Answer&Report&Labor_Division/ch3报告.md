**ch3报告**

实现了**sys_task_info**，用于获取当前任务的信息（运行时长，当前状态和系统调用次数）。

具体实现方法是，在TaskManager里加入get_task_info方法，在TaskControlBlock里维护程序开始时间和syscall次数的数组。

①运行时长：在调度时，通过get_time更新开始时间。info存储的运行时长，由调用时的时间-开始时间得到。

②当前状态：一定是running。

③系统调用次数：syscall数组存放调用次数，通过在mod.rs里的syscall里调用update_syscall_times（在task_manager里实现）。

get_task_info和update_syscall_times的实现都是先获得TaskManagerInner，再获取current_task，最后通过参数，更新对应信息。