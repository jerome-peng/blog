# [PG-无缝自增ID的实现](https://github.com/digoal/blog/blob/master/201610/20161020_02.md)

转至https://github.com/digoal/blog/blob/master/201610/20161020_02.md

postgres=# create table uniq_test(id int primary key, info text);
CREATE TABLE

create or replace function f_uniq(i_info text) returns int as $$
declare
  newid int;
  i int := 0;
  res int;
begin
  loop 
    if i>0 then 
      perform pg_sleep(0.2*random());
    else
      i := i+1;
    end if;

    -- 获取已有的最大ID+1 (即将插入的ID)
    select max(id)+1 into newid from uniq_test;
    if newid is not null then
      -- 获取AD LOCK
      if pg_try_advisory_xact_lock(newid) then
        -- 插入
	insert into uniq_test (id,info) values (newid,i_info);
        -- 返回此次获取到的UID
	return newid;
      else
	-- 没有获取到AD LOCK则继续循环
	continue;
      end if;
    else
      -- 表示这是第一条记录，获取AD=1 的LOCK
      if pg_try_advisory_xact_lock(1) then
	insert into uniq_test (id, info) values (1, i_info);
        return 1;
      else
	continue;
      end if;
    end if;
  end loop;
  
  -- 如果因为瞬态导致PK冲突了，继续调用
  exception when others then
    select f_uniq(i_info) into res;
    return res;
end;
$$ language plpgsql strict;
