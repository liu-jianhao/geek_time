# 23 | MySQL是怎么保证数据不丢的?

## binlog 的写入机制
binlog 的写入逻辑比较简单:事务执行过程中，先把日志写到 binlog cache，事务提交的时候，再把 binlog cache 写到 binlog 文件中。

## redo log 的写入机制
事务在执行过程中，生成 的 redo log 是要先写到 redo log buffer 的。
