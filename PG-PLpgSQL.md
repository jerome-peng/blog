# [PL/pgsql](http://note.youdao.com/noteshare?id=http://note.youdao.com/noteshare?id=c00336936f097adad404e8e387830da8&sub=9DA3E14EEB264F11AE0A217C45CD894D)

1 函数可以使用VARIADIC标记为接受数量不定的参数
CREATE FUNCTION myfunc(VARIADIC arr numeric[]) RETURNS numeric 

2 不要把PL/pgSQL中用来分组语句的BEGIN/END与用于事务控制的同名 SQL 命令弄混。
PL/pgSQL的BEGIN/END只用于分组，不会开始或结束一个事务。
函数和触发器过程总是被执行在由一个外层查询建立的事务中，
它们不能开始或提交那个事务，因为没有环境给它们执行。
不过，包含EXCEPTION子句的块实际上会形成一个子事务，
它可以被回滚而不影响外层事务。

3 等号（=）可以被用来代替 PL/SQL-兼容的 :=

4 复制类型
variable%TYPE
user_id users.user_id%TYPE;

5 STRICT选项
BEGIN
    SELECT * INTO STRICT myrec FROM emp WHERE empname = myname;
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            RAISE EXCEPTION 'employee % not found', myname;
        WHEN TOO_MANY_ROWS THEN
            RAISE EXCEPTION 'employee % not unique', myname;
END;

6  动态表
CREATE OR REPLACE FUNCTION myfun(
    IN v_start character varying,
    IN v_end character varying)
  RETURNS TABLE(o_dept_name character varying, ...) AS
$BODY$

7 小计功能
group by rollup


8 perform


9 With
with t1 as
(delete from basic_login_log where employee_no=v_employee_no::numeric returning *,now())
insert into basic_login_log_his select * from t1;

10 Upsert使用
insert into t values(1,'C++') ON CONFLICT(id) do update set name=EXCLUDED.name；
insert into t values(1,'Java') ON CONFLICT(id) do nothing;

11 row 行构造器
行构造器可以被用来构建存储在一个组合类型表列中的组合值，
或者被传递给一个接受组合参数的函数。
可以比较两个行值，或者用IS NULL或IS NOT NULL测试一个行
例如：
SELECT ROW(1,2.5,'this is a test') = ROW(1, 3, 'not the same'); 
SELECT ROW(table.*) IS NULL FROM table; -- detect all-null rows

12 获得异常的信息
EXCEPTION WHEN others THEN 
      GET STACKED DIAGNOSTICS
          v_state = RETURNED_SQLSTATE, 
          v_msg = MESSAGE_TEXT, 
          v_detail = PG_EXCEPTION_DETAIL, 
          v_hint = PG_EXCEPTION_HINT, v_context = PG_EXCEPTION_CONTEXT; 
 raise notice E'Got exception: state : % message: % detail : % hint : % context: %', v_state, v_msg, v_detail, v_hint, v_context;

使用异常处理来酌情执行INSERT或UPDATE：
 BEGIN
            INSERT INTO db(a,b) VALUES (key, data);
            RETURN;
            EXCEPTION WHEN unique_violation THEN
            -- 什么也不做，并且循环再次尝试 UPDATE
 END;


13 锁
SELECT FOR SHARE加共享行锁
SELECT FOR UPDATE加行排它锁

14 动态 SQL 语句也可以使用format函数来安全地构造。例如：
EXECUTE format('UPDATE tbl SET %I = %L WHERE key = %L', colname, newvalue, keyvalue);
format函数可以和USING子句联合使用：
EXECUTE format('UPDATE tbl SET %I = $1 WHERE key = $2', colname)
   USING newvalue, keyvalue;
这种形式更高效，因为参数newvalue和keyvalue不会被转换成文本。
