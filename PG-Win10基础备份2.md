# Win10基础备份2

PG version 9.5

增加  max_wal_senders=1

此步骤是在配置流复制时使用


主库修改认证配置文件，添加test_rep的replication认证信息：
host    replication     postgres        ::1/128                 trust
host    replication     test_rep        127.0.0.1/32            md5

pg_basebackup -D c:\tcps\pg_base_backup -h localhost -U postgres -Ft -z -P 

postgres=# create table t6(id integer);
CREATE TABLE
postgres=# insert into t6 values(111),(222),(333);
INSERT 0 1

删除data模拟数据丢失(或将此目录改名)
解压基础备份文件，放置到data下

将旧的pg_xlog文件复制到备份下的data目录

建立data/recovery.conf文件（注意文件的权限配置，写权限）
加入以下内容
restore_command = 'copy "C:\\tcps\\pg-archive\\%f" "%p"'
#recovery_target_time='2017-02-20 23:08:58.000000+08'
recovery_target_timeline = 'latest'
