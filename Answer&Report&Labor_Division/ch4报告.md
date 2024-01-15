**ch4报告**

**①重写 sys_get_time 和 sys_task_info**



​	不用修改TaskManager里的方法，只需将传入的虚拟地址转化为物理地址，再调用。



**②mmap 和 munmap 匿名映射**



​	mmap：先判断参数是否合法，再获得当前任务的memory_set。先确认参数范围内的虚拟页号对应的页是否已经被分配过，如果都未分配，根据port参数，得到地址区间的权限，再调用封装好的insert_framed_area，将页表插入memory_set；如果分配过，则返回-1。

​	munmap：类似mmap，但是memory_set没有实现unmap，需要自己实现。对memory_set里的每个map_area，将其与unmap的范围重合的部分调用MapArea的unmap_one即可。