# PG函数自治事务

## 转https://github.com/digoal/blog/blob/master/201602/20160203_02.md

因为PostgreSQL的函数是作为一个事务来处理的，要么全部提交，要么全部回滚，

除了exception，每个exception是一个子事务。

因此使用exception可以达到自治事务的目的。

使用并行block和嵌套block，来控制子事务层级。

输入参数为block1, block2.1, block2.2, block3.1 。

这些参数代表执行在哪个block出错，出错时对应层级的block的exception会捕获错误，同时处理，然后跳到下一个block继续执行。

如果是外层的block出错，内层还没有被执行的block就没机会执行了。

根据业务需求，调整block层级或嵌套层级，达到目的。

create or replace function ft(err_level text) returns void as $$  
declare  
begin -- block level 1  
  raise notice 'block level 1';  
  if (err_level='block1') then  
    raise exception '%', err_level;  
  end if;  
  
  begin -- block level 2.1  
    raise notice 'block level 2.1';  -- 请用业务处理SQL代替  
    if (err_level='block2.1') then  
      raise exception '%', err_level;  
    end if;  
  
    begin -- block level 3.1  
      raise notice 'block level 3.1';  
      if (err_level='block3.1') then  
        raise exception '%', err_level;  
      end if;  
      exception when others then  -- you can write catchup any ERROR CODE or ERROR STATE.  
        raise notice 'end block level 3.1';  
    end; -- end block level 3.1  
  
    exception when others then  -- you can write catchup any ERROR CODE or ERROR STATE. 回滚block 2.1的业务处理SQL  
      raise notice 'end block level 2.1';  
  end; -- end block level 2.1  
  
  begin -- block level 2.2  
    raise notice 'block level 2.2';  
    if (err_level='block2.2') then  
      raise exception '%', err_level;  
    end if;  
    exception when others then  -- you can write catchup any ERROR CODE or ERROR STATE.  
      raise notice 'end block level 2.2';  
  end; -- end block level 2.2  
  
  exception when others then  -- you can write catchup any ERROR CODE or ERROR STATE.  
    raise notice 'end block level 1';  
end; -- end block level 1  
$$ language plpgsql;  
