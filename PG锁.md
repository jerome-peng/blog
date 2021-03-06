# [PG锁](http://note.youdao.com/noteshare?id=20fc7138f7fba570dfb6a35d5214dae9&sub=C2B030FD939842E4A617D7D120187111)

## 锁类型
表锁：有8种锁
行锁：2种锁

## 8种表锁
ACCESS SHARE
ROW SHARE
ROW EXCLUSIVE
SHARE UPDATE EXCLUSIVE
SHARE
SHARE ROW EXCLUSIVE
EXCLUSIVE
ACCESS EXCLUSIVE

## 开始只有两种锁
SHARE：共享锁，或可以理解为读锁，多个事务都可以同时加上此锁。但任何一个事务加上此锁后，就不能修改数据。
EXCLUSIVE：可以理解为排它锁，或写锁。加上此锁后，再修改数据。

## 多版本导致需要增加新的锁
改了一行数据，实际上并没有改原先那行数据，而是先复制出一个新行，然后在新行上改，导致修改数据的同时可以读数据


## 多版本增加的锁
ACCESS SHARE：表明加上这个锁，即使正在的修改数据的情况下，还是让读数据
ACCESS EXCLUSION：即使有多版本的功能，也不允许读数据

多版本之后：以前的“SHARE”锁，意思还是与原先一样，加了这个锁下，就不能对表作任何的变更；同样以前的“EXCLUSIVE”锁还叫“EXCLUSIVE”锁，意思与原来的意思也差不多，就是除可以允许多版本的读，其它的写操作都被禁止。

## 意向锁的出现
处理行级锁与表级锁之间的冲突而增加的表级锁
修改表中的一行数据时，需要先在表上加一种锁，表示即将在表的部分行上加共享锁或排它锁
ROW SHARE
ROW EXCLUSIVE

## 意向锁的特点
意向锁之间总是不会发生冲突的，因为它们都只是“有意”要做什么，还没有真做
意向锁与其它非意向锁之间的关系与普通锁与普通锁之间的关系是相同。举例：“X”与“X”锁是冲突的，所以“IX”锁与“X”是冲突的；“X”与“S”锁是冲突的，所以“IX”锁与“S”也是冲突的；“S”与“S”锁是不冲突的，所以“IS”与“S”也是不冲突的。
如果我们把共享锁“SHARE”简写为“S”，排它锁“EXCLUSIVE”简写为“X”，把“ROW SHARE”简写为“IS”，把“ROW EXCLUSIVE”简写为“IX”

## 需要比意向锁更严格的锁：第七种锁
意向排它锁“IX”与自己之间是不会冲突的，这时我们可能就需要一种稍严格的锁，就是自己与自己不兼容的锁，和其它锁的冲突情况下与“IX”相同。 如VACUUM、CREATE INDEX CONCURRENTLY这些命令就需要这种锁，这些命令不允许在同一个表上并发执行，但允许其它的更新命令执行（即允许表上再加IX锁）。
这种锁就叫“SHARE UPDATE EXCLUSIVE”。


## 最后一种锁：意向锁IX与S锁组合的锁
SHARE ROW EXCLUSIVE
同时加了“S”锁和“IX”锁的结果
PostgreSQL命令都不会自动请求这个锁模式，也就是PostgreSQL内部目前不会使用这种锁


## 2 种行锁
行共享锁或称之为读锁
行排它锁或称之为写锁
当执行更新操作时，自动加行排它锁
SELECT FOR SHARE加共享行锁
SELECT FOR UPDATE加行排它锁

## 死锁及防范
四个必要条件
互斥条件：同时只能有一个事务持有
请求和保持条件：事务已经至少持有了一把排它锁，但又提出了新的排它锁请求
不剥夺条件
环路等待条件

## 防范
相同的加锁顺序
申请第二把锁时，如果无法申请到，则释放第一把锁：LOCK TABLE NOWAIT
数据库可以自动检测出死锁，所以应用也可以通过捕获死锁异常来处理死锁。但这个方法不是一个很好的方法，因为数据库检测死锁需要一定的代价，可能会导致应用程序过久的持有了排它锁，导致系统的并发处理能力下降。
尽量短的持有排它锁

## pg_locks视图

## 锁类型不只有表锁和行锁两种
还有一些其它数据库对象上会出现锁，如索引、extend、page、tuple、transactionid、virtualxid、object、userlock或 advisory
除了一些数据库对象上有锁，还有transactionid和virtualxid的锁

参考 唐成-PostgreSQL锁分析
