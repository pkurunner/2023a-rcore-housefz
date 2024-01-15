ch8

    模仿文档中的算法实现。

    mutex：在ProcessControlBlockInner中维护mutex_allocation(可算出available向量)和mutex_need，在调用lock时先更新need再运行算法检测，在新建进程、线程、mutex和调用lock、unlock时更新维护的数据。
    
    semaphore：与mutex实现基本相同