# PG9.6 bloom索引

一个索引支撑任意列组合查询，如果使用其他的索引方法，任意条件组合查询，需要为每一种组合创建一个索引来支持。

而使用bloom索引方法，只需要创建一个索引就够了。

create index idx on test using bloom(id);  

## 参考 
https://github.com/digoal/blog/blob/master/201605/20160523_01.md
http://www.postgresql.org/docs/9.6/static/bloom.html
