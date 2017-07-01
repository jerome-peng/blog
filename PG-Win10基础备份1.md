# Win10基础备份1

PG version 9.5

SELECT pg_start_backup('2017022102');
 \! zip -r C:\tcps\pg-full-bak\fullbackup2017022102.zip C:\tcps\pg1\data -x C:\tcps\pg1\data\postmaster.pid ;
SELECT pg_stop_backup();


postgres=# create table t2(id integer);
CREATE TABLE
postgres=# insert into t2 values(11),(22),(33);
INSERT 0 1


删除data模拟数据丢失(或将此目录改名)

解压基础备份文件，放置到data下

将pg_xlog文件复制到data目录

建立data/recovery.conf文件（注意文件的权限配置，写权限）
加入以下内容
restore_command = 'copy "C:\\tcps\\pg-archive\\%f" "%p"'
#recovery_target_time='2017-02-20 23:08:58.000000+08'
recovery_target_timeline = 'latest'
