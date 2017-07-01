# [PG中json的使用](http://note.youdao.com/noteshare?id=06f142c7ba6af524c140b86744baecf2&sub=3326A58D38744FA193E1C9F55CD07A7B)

查询返回JSON字符串
student=> select row_to_json(b.*) from basic_dept b where b.dept_no=0;   
                             row_to_json                                                         
 {"dept_no":0,"dept_code":"0","dept_name":"Service Center","dept_level":0,
"parent_dept_no":-1,"modify_date":"2016-09-30T14:34:16.265219",
"is_run":1,"city_no":1003,"is_issue":1}

不需要一整行数据（如只要部门号和部门名称）
student=> select row_to_json(b.*) from (select dept_no,dept_name from basic_dept where dept_no=0) b;  
                row_to_json                 
--------------------------------------------
 {"dept_no":0,"dept_name":"Service Center"}

或者用with
student=> with t as (select dept_no,dept_name from basic_dept where dept_no=0)
student-> select row_to_json(t.*) from t ;
                row_to_json                 
--------------------------------------------
 {"dept_no":0,"dept_name":"Service Center"}

查询有多行，形成json数组
with running_dept as (select dept_no,dept_name from basic_dept where dept_no>0)
select row_to_json(b.*) from
(select array_to_json(array(select row_to_json(running_dept.*) from running_dept),false) as running_dept) b ;
                         row_to_json  
 {"running_dept":[{"dept_no":291,"dept_name":"test_dept_10"},{"dept_no":290,"dept_name":"test_dept_11"},
{"dept_no":281,"dept_name":"test_dept_12"},
{"dept_no":275,"dept_name":"test_dept_13"},
{"dept_no":284,"dept_name":"test_dept6"},
{"dept_no":273,"dept_name":"DEPT_2"}]}

返回卡账户基本信息及该卡账户最近三笔充值记录
with cardInfo as (select card_type,supply_money,paid,card_money from card_account where card_no='2001100300000121' ),
supplyInfo as (select ss_times,to_char(supply_time,'yyyy-mm-dd hh24:mi:ss') as supply_time,supply_money  from issue_card_supply where card_no='2001100300000121' order by supply_time desc limit 3)
SELECT row_to_json(x.*) from(
    select  c.*,d.*  from cardInfo c,
     (SELECT array_to_json(array(select row_to_json(supplyInfo.*)  from supplyInfo),false) as supplyInfo) d 
) x


JSON的操作
create table ic_ladder_test (vno character varying, up_data jsonb);
insert into ic_ladder_test values('2016120701','{"key1":"val1","key2":100,"key3":100.1}');
student=> select * from ic_ladder_test ;
    vno     |                   up_data                    
------------+----------------------------------------------
 2016120701 | {"key1": "val1", "key2": 100, "key3": 100.1}

l增
update ic_ladder_test set up_data = up_data || '{"station name":"bus station"}' ;
student=> select * from ic_ladder_test ;
    vno     |                                   up_data                                   
------------+-----------------------------------------------------------------------------
 2016120701 | {"key1": "val1", "key2": 100, "key3": 100.1, "station name": "bus station"}

l删
student=> update ic_ladder_test set up_data = up_data - 'key3';
UPDATE 1
student=> select * from ic_ladder_test ;
    vno     |                           up_data                            
------------+--------------------------------------------------------------
 2016120701 | {"key1": "val1", "key2": 100, "station name": "bus station"}

l改
insert into ic_ladder_test values('2016120702','{"key1":"val1","key2":200,"key3":200.1}');
update ic_ladder_test set up_data =jsonb_set(up_data, '{key2}','300', true) where vno='2016120702';
update ic_ladder_test set up_data =jsonb_set(up_data, '{key2}','"300"', true) where vno='2016120702';
注意：'{up_data}'不能是'up_data'

l查
student=> select * from ic_ladder_test where up_data @>'{"key2":"300"}';
    vno     |                             up_data                              
------------+------------------------------------------------------------------
 2016120702 | {"key1": "val1", "key2": "300", "key3": 200.1, "up_data": "123"}

JSON数组的操作
l插入数组：
insert into ic_ladder_test values('2016120703',
'[{"s1-s2":"sname1-->sname2","price":10,"flag":"up"},
{"s2-s3":"sname2-->sname3","price":20,"flag":"up"},
{"s1-s3":"sname1-->sname3","price":30,"flag":"up"},
{"s1-s2":"sname1-->sname2","price":10,"flag":"down"},
{"s2-s3":"sname2-->sname3","price":20,"flag":"down"},
{"s1-s3":"sname1-->sname3","price":30,"flag":"down"}]');

l增
update ic_ladder_test set up_data = '{"s0-s1":"sname0-->sname1","price":30,"flag":"up"}' || up_data where vno = '2016120703'; -- 新值在前面

l删
update ic_ladder_test set up_data = up_data -0 where vno = '2016120703';
update ic_ladder_test set up_data = up_data -1 where vno = '2016120703';
update ic_ladder_test set up_data = up_data #-'{0,price}' where vno = '2016120703';

l改
update ic_ladder_test set up_data=
jsonb_set(up_data,'{0}','{"s00-s01":"sname00-->sname01","price":1,"flag":"up"}',false) where vno = '2016120703';

update ic_ladder_test set up_data=
jsonb_set(up_data,'{0,price}','200',false) where vno = '2016120703';

update ic_ladder_test set up_data=
jsonb_set(up_data,'{0,price}','"200"',false) where vno = '2016120703';


select * from ic_ladder_test where up_data @>'[{"price":50}]'::jsonb;
 2016120703 | [{"flag": "up", "price": 50, "s00-s01": "sname00-->sname01"}, {"flag": "up", "price": 20, "s2-s3": "sname2-
->sname3"}, {"flag": "up", "price": 30, "s1-s3": "sname1-->sname3"}, {"flag": "down", "price": 10, "s1-s2": "sname1-->sna
me2"}, {"flag": "down", "price": 20, "s2-s3": "sname2-->sname3"}, {"flag": "down", "price": 30, "s1-s3": "sname1-->sname3
"}]

l查
（找出符合条件“数组中第0个元素的price<100的） 
select * from ic_ladder_test where cast(up_data ->0->>'price' as int)<100;
select cast(up_data ->0->>'price' as int) tt from ic_ladder_test where cast(up_data ->0->>'price' as int)<100;
