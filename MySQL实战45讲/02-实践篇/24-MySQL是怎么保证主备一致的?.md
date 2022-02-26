# 24 | MySQL是怎么保证主备一致的?

备库 B 跟主库 A 之间维持了一个长连接。主库 A 内部有一个线程，专门用于服务备库 B 的这个长连接。一个事务日志同步的完整过程是这样的:
1. 在备库 B 上通过 change master 命令，设置主库 A 的 IP、端口、用户名、密码，以及 要从哪个位置开始请求 binlog，这个位置包含文件名和日志偏移量。
2. 在备库 B 上执行 start slave 命令，这时候备库会启动两个线程，就是图中的 io_thread 和 sql_thread。其中 io_thread 负责与主库建立连接。
3. 主库 A 校验完用户名、密码后，开始按照备库 B 传过来的位置，从本地读取 binlog， 发给 B。
4. 备库 B 拿到 binlog 后，写到本地文件，称为中转日志(relay log)。
5. sql_thread 读取中转日志，解析出日志里的命令，并执行。

## binlog 的三种格式对比
一种是 statement，一种是 row。可能你在其他资料上还会看到有第三种格式，叫作 mixed，其实它就是前两种格式 的混合。

因为有些 statement 格式的 binlog 可能会导致主备不一致，所以要使用 row 格式。
但 row 格式的缺点是，很占空间。比如你用一个 delete 语句删掉 10 万行数据，用 statement 的话就是一个 SQL 语句被记录到 binlog 中，占用几十个字节的空间。但如 果用 row 格式的 binlog，就要把这 10 万条记录都写到 binlog 中。这样做，不仅会占 用更大的空间，同时写 binlog 也要耗费 IO 资源，影响执行速度。

MySQL 就取了个折中方案，也就是有了 mixed 格式的 binlog。mixed 格式的意 思是，MySQL 自己会判断这条 SQL 语句是否可能引起主备不一致，如果有可能，就用 row 格式，否则就用 statement 格式。