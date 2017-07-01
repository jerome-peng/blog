# [WAL日志文件分析](http://note.youdao.com/noteshare?id=041ccb498603cc49f97334bd11205eed&sub=31A45DB2344D4A70A4E08453207D5569)

Log Sequence Number：WAL日志的绝对位置

PostgreSQL 的 WAL 日志文件在 pg_xlog 目录下，一般情况下，每个文件为 16M 大小

文件名称为 16 进制的 24 个字符组成，每 8 个字符一组，每组的意义如下： 

 00000001 00000000 000000BC   
  -------- -------- --------      
  时间线    LogId    LogSeg 

时间线：英文为timeline，是以1开始的递增数字，如1,2,3...
LogId：32bit长的一个数字，是以0开始递增的数字，如0,1,2,3...实际为LSN的高32bit
LogSeg：32bit长的一个数字，是以0开始递增的数字，如0,1,2,3...
LogSeg是LogSeg是LSN的低32bit的字节的值再除以WAL文件大小（16M）的结果。注意：当LogId为0时，LogSeg是从1开始的。

需要注意的是，数据库刚建好时，LogSeg 第一次是从 1 开始的数字，到达一个最大的数字后，会重头开始，但以后再从头开始时，不再从 1 开始，而是从 0 开始。 

LogSeg 的值是从 00000000 到 000000FF 后，就重新从 00000000 开始了，并不会出现 00000100 这样的数值，也就是 LogSeg 的 8 个字符中， 前 6 个字符始终是0。

原来 LogSeg 是按文件递增，每增加一个文件，LogSeg 就增加 1，而每个 WAL 文件的大小是 16M，LogSeg 是 LSN 的低 32bit 的字节的值再除以16M的结果，这样 4G/16M 结果是 256， 所以 LogSeg 最大的值为 256，即 16 进制的 0xFF，这样就解释了LogSeg的前 6 字节都是零的原因。 

另原本 LSN 起始位置可以从 0 开始，但为了表示一些无效的位置，LSN 的起始值并不是从 0 开始，也不是从 1 开始，而是跳过了一个 WAL 文件大小，即 16M 的位置开始，这样第一次 时 LogSeg 是从 1 开始的，而不是从 0 开始了。 

我们知道了 WAL 日志文件名中的意义，但很多时候我们会发现在 PostgreSQL 中会用两个十 六进制的数字中间用斜杠“/”分隔表示 WAL 日志位置，即表示 LSN，如函数 pg_current_xlog_location、pg_current_xlog_insert_locatione 及 pg_start_backup 的返回结果。
student=> select pg_current_xlog_location(), pg_current_xlog_insert_location(); 
 pg_current_xlog_location | pg_current_xlog_insert_location 
--------------------------+---------------------------------
 2/694C58A8               | 2/694C58A8

同时我们在主库上通过查询视图 pg_stat_replication 来看 replication 的状态时，一些字段也 显示成这样的格式： 

以上这些表示 WAL 日志位置的方法，就是用两个 32bit 的数字来表示 LSN，前一个数字 LSN 的高 32bit，而另一个数字则是 LSN 的低 32bit。在 PostgreSQL9.4 之后，增加了一个专门的类型 pg_lsn 用于表示 WAL日志的位置，在 PostgreSQL9.4 之前，则用字符串来表示这个位置。 

函数 pg_current_xlog_location 返回 2/694C58A8，如果当前 timeline 为 1， 那么当前的 WAL 日志文件名是什么呢？ 

logId 就是“2/694C58A8”中的第一个数字，即 2 logSeg 就是“2/694C58A8”中的第二个数字除以 16M 的大小，即 694C58A8 除以 16M，而 16M 相当于 2 的 24 次方，相当于十六进制数“694C58A8”右移 6 位，即“694C58A8”中的 最高两位“69” 那么根据 WAL 文件的格式 timeline+logId+logSeg，则相当于:“00000001”+“00000002”+ “00000069”，即为：“000000010000000200000069” 写的位置是在文件“000000010000000200000069”中的偏移量是多少呢？实际上是在 “2/694C58A8”中第二个数字“694C58A8”后六位“4C58A8”，换算成十进制为“5003432”。 

当然 PostgreSQL 已准备了函数 pg_xlogfile_name_offset 帮我们做以上的转换，如下所示： 

student=> select pg_xlogfile_name_offset('2/694C58A8');
      pg_xlogfile_name_offset       
------------------------------------
 (000000010000000200000069,5003432)
 
 参考文档<<WAL日志文件的秘密>>
