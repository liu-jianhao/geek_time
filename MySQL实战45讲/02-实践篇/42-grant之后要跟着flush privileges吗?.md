# 42 | grant之后要跟着flush privileges吗?

grant 语句会同时修改数据表和内存，判断权限的时候使用的是内存数据。因此，规范地使 用 grant 和 revoke 语句，是不需要随后加上 flush privileges 语句的。

flush privileges 语句本身会用数据表的数据重建一份内存权限数据，所以在权限数据可能 存在不一致的情况下再使用。