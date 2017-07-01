# [PG多版本并发控制](http://note.youdao.com/noteshare?id=e7916b9df7db0b96f7a713e3c5305550&sub=18744DA744214E5CA231700DF5B484A4)

多版本并发控制(Multiversion Concurrency Control, MVCC)

每行上有xmin和xmax两个系统字段

当插入一行数据时，将这行上的xmin设置为当前的事务id，而xmax设置为0

当更新一行时，实际上是插入新行，把旧行上的xmax设置为当前事务id，新插入行的xmin设置为当前事务id，新行的xmax设置为0

当删除一行时，把当前行的xmax设置为当前事务id

当读到一行时，到commitlog中查询xmin和xmax对应的事务状态是否是已提交还是回滚了，就能判断出此行对当前行是否是可见
