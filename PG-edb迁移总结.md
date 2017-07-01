# [edb迁移至PG总结](http://note.youdao.com/noteshare?id=9127644e626b25f230d40f3ef3f0890b&sub=A2D044B2279347608328B777EC7BBCE4)

## 时间：2016年8月30号

1 修改表结构时需要先删除视图引用

2 pg没有存储过程，所有的块操作都通过函数完成
   业务修改函数调用方式

3 pg里没有rollback语句不会自动rollback，需要应用里触发异常

4 没有手动commit

5 外联接需要采用 left out join 而不是 oracle 里的 a.id=b.id(+) 这种方式

6 修改表时,表名不能设置别名,如
     update basic_employees b set b.name='XX'  

7 sysdate->now()   current_date  current_timestamp

8 NVL -> coalesce() 

9 decode -> case…when… end

10  数据类型比较时类型一定要匹配

11 转数据类型用::数据类型或者cast,
     如to_number() 改用::numeric(number--> numeric)
       to_char()需要指定转换的格式，如to_char(3,'99') 
       to_numer()需要指定转换的格式
       to_date() 只能到date   to_timestamp()
       add_months    date+interval

12 rownum -> row_number() over()

13 startwith…connectby -> WITH RECURSIVE 

14 'a'||null （O结果是'a'，P结果是null) -> a||coalesce(b,'') 

15 sys_guid() -> uuid_generate_v4()  
     create extension "uuid-ossp";

16 %FOUND -> found    %NOTFOUND -> not found

17 字段别名加上 as

18 子查询一定要用别名 

19  dual: create view dual as select 1;

20  触发器，需要先建立触发器函数，再创建触发器
