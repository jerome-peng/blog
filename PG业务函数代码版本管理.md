# [PG业务函数代码版本管理](http://note.youdao.com/noteshare?id=4dddacb3ddb3018255d611e60a13a5bb&sub=4797C4FE3E81416AB3C4A2B8133CB009)
## 1. 创建存储函数代码的表
create table public.svn_func(  
  id serial8 primary key,  -- 序列  
  tx int8, -- 事务号  
  objid oid, -- 函数唯一标示 pg_proc.oid  
  object_type text, -- 类型  
  schema_name text, -- schema name  
  object_identity text, -- 全长对象名: schema_name.object_name  
  in_extension bool, -- 对象是否属于extension  
  crt_time timestamp, -- DDL时间  
  content text  -- DDL翻译成文本  
);  
## 2. 创建事件触发器函数
create or replace function public.push_to_svn_func() returns event_trigger as $$  
declare  
  r record;  
begin  
  for r in SELECT * FROM pg_event_trigger_ddl_commands() LOOP  
    insert into public.svn_func(tx, objid, object_type, schema_name, object_identity, in_extension, crt_time, content)  
      values   
       (  
          txid_current(),  
	  r.objid,   
          r.object_type,  
          r.schema_name,  
          r.object_identity,  
          r.in_extension,  
          now(),  
          pg_get_functiondef(r.objid)  
	);  
  end LOOP;  
end;  
$$ language plpgsql strict;  
## 3. 创建事件触发器
student=# create event trigger et1 on ddl_command_end  when TAG in ('create function') execute procedure push_to_svn_func();  
CREATE EVENT TRIGGER

grant all on public.svn_func to eafc,ebas;
grant all on public.svn_func_id_seq to ebas,eafc

revoke update on public.svn_func from ebas,eafc;
revoke delete on public.svn_func from ebas,eafc;
revoke truncate on public.svn_func from ebas,eafc;
revoke references on public.svn_func from ebas,eafc;
revoke trigger on public.svn_func from ebas,eafc;
## 4. 测试
### 4.1 创建函数(eafc)
create or replace function f123(id int) returns int as $$                                                           
declare  
begin  
return id+1;  
end;  
$$ language plpgsql strict; 
CREATE FUNCTION
### 4.2 创建同名，但是参数不同的函数(eafc)
create or replace function f123(id int, diff int) returns int as $$  
declare  
begin  
return id+diff;  
end;  
$$ language plpgsql strict;  
### 4.3 创建完全相同的函数，写入不同的SCHEMA(ebas)
create or replace function ebas.f123(id int, diff int) returns int as $$  
declare  
begin  
return id+diff;  
end;  
$$ language plpgsql strict;  
### 4.4 覆盖创建原有函数
create or replace function ebas.f123(id int, diff int) returns int as $$  
declare  
begin  
return id+diff;  
end;  
$$ language plpgsql strict;
### 4.5 查看函数内容记录
select * from svn_func;

参考https://github.com/digoal/blog/blob/master/201703/20170305_01.md
