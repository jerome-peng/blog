# [CentOS7源码安装postgresql10](http://note.youdao.com/noteshare?id=cd740aea572cb016f5ead126006c9d15&sub=3C7C1365459341F296BE2B88FD56A269)


环境CentOS7 X86_64

https://yum.postgresql.org/news10snapshot-ready-for-testing.php

创建用户
useradd postgres
password postgres
密码为logo

vi /etc/yum.repos.d/CentOS-Base.repo 
[base] 和[updates] 区段之间添加： 
exclude=postgresql*

yum localinstall pgdg-centos10-10-1.noarch.rpm
yum list postgresql*
yum install postgresql10-server.x86_64

mkdir -p /var/lib/pgsql/10/data
/usr/pgsql-10/bin/postgresql-10-setup initdb
shell-init: error retrieving current directory: getcwd: cannot access parent directories: No such file or directory
Initializing database ... OK

切换到postgres用户执行
vi /var/lib/pgsql/10/data/postgresql.conf
修改端口为9432

/usr/pgsql-10/bin/pg_ctl -D /var/lib/pgsql/10/data -l /var/lib/pgsql/10/pg.log start &

[postgres@dw-gp-1 ~]$ /usr/pgsql-10/bin/pg_ctl -D /var/lib/pgsql/10/data status
pg_ctl: server is running (PID: 10743)
/usr/pgsql-10/bin/postgres "-D" "/var/lib/pgsql/10/data"

/usr/pgsql-10/bin/psql -p 9432  
psql (10beta1)
Type "help" for help.

postgres=# 




