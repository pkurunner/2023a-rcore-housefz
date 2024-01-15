ch6
 - 迁移之前系统调用：一样，除了将spawn改为用open_file实现
 - fstat：反转block_id和block_offset计算过程得inode_id，遍历所有文件得nlink
 - sys_linkat：old_name查inode_id，和new_name新建DirEntry并写入根节点
 - unlink：硬链接＞1则将对应名称处写入空DirEntry。=1则回收inode，再写入空DirEntry