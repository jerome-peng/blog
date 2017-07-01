# [Centos7.3+pipline安装](http://note.youdao.com/noteshare?id=97f4abf3cb0c8a234fe574266c7b32d6&sub=312F1FE54481478DAEB9AC9CDB8D3E2F)

## 部署依赖
安装 zeromq
http://zeromq.org/intro:get-the-software
wget https://github.com/zeromq/libzmq/releases/download/v4.2.2/zeromq-4.2.2.tar.gz    
tar -zxvf zeromq-4.2.2.tar.gz      
cd zeromq-4.2.2     
./configure    
make    
make install    
    
vi /etc/ld.so.conf    
/usr/local/lib     
ldconfig   
 
## 下载pipelinedb 
wget https://github.com/pipelinedb/pipelinedb/releases/download/0.9.7u4/pipelinedb-0.9.7u4-centos7-x86_64.rpm

rpm -ivh --prefix=/opt/pipelinedb pipelinedb-0.9.7u4-centos7-x86_64.rpm

cd /opt/pipelinedb
mkdir dbdata
chown gpadmin.gpadmin dbdata/
chmod 755 dbdata/

pipeline-init -D /opt/pipelinedb/dbdata -U gpadmin --locale=C

## 修改pipelinedb.conf 打开监听和修改端口为1432
/opt/pipelinedb/bin/pipeline-ctl -D /opt/pipelinedb/dbdata -l piplinedb_log start &

/opt/pipelinedb/bin/pipeline-ctl -D /opt/pipelinedb/dbdata status

/opt/pipelinedb/bin/psql -h localhost -p 1432 -d pipeline -c "ACTIVATE" 

# /opt/pipelinedb/bin/psql -h localhost -p 1432 -U gpadmin pipeline
