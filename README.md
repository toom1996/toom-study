# Redis 数据持久化

+ AOF(Append Only File)
    + AOF持久化功能的实现可以分为命令追加、文件写入、文件同步三个步骤。
        + 命令追加：
            + 当AOF持久化功能打开时，服务器在执行完一个写命令之后，会以协议格式将被执行的写命令追加到服务器状态的aof_buf缓冲区的末尾。
            
        + AOF文件的写入与同步：
            + 每当服务器常规任务函数被执行、 或者事件处理器被执行时， aof.c/flushAppendOnlyFile 函数都会被调用， 这个函数执行以下两个工作：
              WRITE：根据条件，将 aof_buf 中的缓存写入到 AOF 文件。

              SAVE：根据条件，调用 fsync 或 fdatasync 函数，将 AOF 文件保存到磁盘中。
              
              两个步骤都需要根据一定的条件来执行， 而这些条件由 AOF 所使用的保存模式来决定。
              
              Redis 目前支持三种 AOF 保存模式，它们分别是：
              
              AOF_FSYNC_NO ：不保存。
                + 在这种模式下， 每次调用 flushAppendOnlyFile 函数， WRITE 都会被执行， 但 SAVE 会被略过。
                  
                  在这种模式下， SAVE 只会在以下任意一种情况中被执行：
                  
                  Redis 被关闭
                  
                  AOF 功能被关闭
                  
                  系统的写缓存被刷新（可能是缓存已经被写满，或者定期保存操作被执行）
                  
                  这三种情况下的 SAVE 操作都会引起 Redis 主进程阻塞。
              
              AOF_FSYNC_EVERYSEC ：每一秒钟保存一次。
                + ![avatar](./01.png)
              
              AOF_FSYNC_ALWAYS ：每执行一个命令保存一次。

        + 