# [postgis2.3.2安装](http://note.youdao.com/noteshare?id=274597a7297cb46ad2c7c0a16b7ef63d&sub=76754038F1D34FE4BDE2C6B42408DC72)

PG version 9.5.7

安装参考 http://blog.csdn.net/clyhs2009/article/details/7920830

一、在postgis官网下载安装版本
http://postgis.net/source/

二、进入目录，执行
需要前提安装Proj4、GEOS、LibXML2、GDAL、json-c(未安装)
如果系统中没有安装则安装。

rpm -qa libxml2
libxml2-2.7.6-17.el6_6.1.x86_64

wget http://download.osgeo.org/geos/geos-3.6.1.tar.bz2(编译时间较长)
wget http://download.osgeo.org/proj/proj-4.9.3.tar.gz
wget http://download.osgeo.org/gdal/2.2.0/gdal-2.2.0.tar.gz(编译时间较长)

/etc/profile
export PGHOME=/opt/PostgreSQL/9.5
export PROJ_HOME=/usr/local/proj
export GEOS_HOME=/usr/local/geos
export GDAL_HOME=/usr/local/gdal

export MAVEN_HOME=/usr/local/apache-maven-3.3.9
export JAVA_HOME=/opt/java/jdk1.8.0_111
export PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$MAVEN_HOME/bin:$PGHOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
export LD_LIBRARY_PATH=$PGHOME/lib:$PROJ_HOME/lib:$GEOS_HOME/lib:$GDAL_HOME/lib

cd proj-4.9.3
./configure --prefix=/usr/local/proj
make  make install

cd geos-3.6.1
./configure --prefix=/usr/local/geos
make make install

cd gdal-2.2.0
./configure --prefix=/usr/local/gdal
make make install

tar zxvf postgis-2.3.2.tar.gz

cd postgis-2.3.2

./configure --with-pgconfig=/opt/PostgreSQL/9.5/bin/pg_config --with-geosconfig=/usr/local/geos/bin/geos-config --with-projdir=/usr/local/proj --with-gdalconfig=/usr/local/gdal/bin/gdal-config

编译过程截取最后几行输出
enabling PostgreSQL extension support...
configure: creating ./config.status
config.status: creating GNUmakefile
config.status: creating extensions/Makefile
config.status: creating extensions/postgis/Makefile
config.status: creating extensions/postgis/postgis.control
config.status: creating extensions/postgis_sfcgal/Makefile
config.status: creating extensions/postgis_sfcgal/postgis_sfcgal.control
config.status: creating extensions/postgis_topology/Makefile
config.status: creating extensions/postgis_topology/postgis_topology.control
config.status: creating extensions/postgis_tiger_geocoder/Makefile
config.status: creating extensions/postgis_tiger_geocoder/postgis_tiger_geocoder.control
config.status: creating extensions/address_standardizer/Makefile
config.status: creating extensions/address_standardizer/address_standardizer.control
config.status: creating extensions/address_standardizer/address_standardizer_data_us.control
config.status: creating liblwgeom/Makefile
config.status: creating liblwgeom/cunit/Makefile
config.status: creating liblwgeom/liblwgeom.h
config.status: creating libpgcommon/Makefile
config.status: creating libpgcommon/cunit/Makefile
config.status: creating postgis/Makefile
config.status: creating postgis/sqldefines.h
config.status: creating loader/Makefile
config.status: creating loader/cunit/Makefile
config.status: creating topology/Makefile
config.status: creating topology/test/Makefile
config.status: creating regress/Makefile
config.status: creating doc/Makefile
config.status: creating doc/Makefile.comments
config.status: creating doc/html/image_src/Makefile
config.status: creating utils/Makefile
config.status: creating raster/Makefile
config.status: creating raster/rt_core/Makefile
config.status: creating raster/rt_pg/Makefile
config.status: creating raster/loader/Makefile
config.status: creating raster/test/Makefile
config.status: creating raster/test/cunit/Makefile
config.status: creating raster/test/regress/Makefile
config.status: creating raster/scripts/Makefile
config.status: creating raster/scripts/python/Makefile
config.status: creating postgis_config.h
config.status: creating raster/raster_config.h
config.status: executing libtool commands
config.status: executing po-directories commands

  PostGIS is now configured for x86_64-pc-linux-gnu

 -------------- Compiler Info ------------- 
  C compiler:           gcc -g -O2
  SQL preprocessor:     /usr/bin/cpp -traditional-cpp -w -P

 -------------- Dependencies -------------- 
  GEOS config:          /usr/local/geos/bin/geos-config
  GEOS version:         3.6.1
  GDAL config:          /usr/local/gdal/bin/gdal-config
  GDAL version:         2.2.0
  PostgreSQL config:    /opt/PostgreSQL/9.5/bin/pg_config
  PostgreSQL version:   PostgreSQL 9.5.7
  PROJ4 version:        49
  Libxml2 config:       /usr/bin/xml2-config
  Libxml2 version:      2.7.6
  JSON-C support:       no
  PCRE support:         no
  PostGIS debug level:  0
  Perl:                 /usr/bin/perl

 --------------- Extensions --------------- 
  PostGIS Raster:       enabled
  PostGIS Topology:     enabled
  SFCGAL support:       disabled
  Address Standardizer support:       disabled

 -------- Documentation Generation -------- 
  xsltproc:             /usr/bin/xsltproc
  xsl style sheets:     
  dblatex:              
  convert:              
  mathml2.dtd:          http://www.w3.org/Math/DTD/mathml2/mathml2.dtd

make && make install

确认postgis组间已安装
ll /opt/PostgreSQL/9.5/share/postgresql/extension/postgis*

创建一个空间测试库
psql -U postgres 
postgres=# create database gis template template0 encoding 'UTF8';
CREATE DATABASE

在空间测试库中加载postgis 和 postgis_topology 模块
使用超级用户创建这两个模块
gis=# create extension postgis;
ERROR:  could not load library "/opt/PostgreSQL/9.5/lib/postgresql/postgis-2.3.so": libgeos_c.so.1: cannot open shared object file: No such file or directory

ldd /opt/PostgreSQL/9.5/lib/postgresql/postgis-2.3.so
	linux-vdso.so.1 =>  (0x00007fff3dd55000)
	libgeos_c.so.1 => /usr/local/geos/lib/libgeos_c.so.1 (0x00007ffd5e122000)
	libproj.so.12 => /usr/local/proj/lib/libproj.so.12 (0x00007ffd5deb7000)
	libxml2.so.2 => /opt/PostgreSQL/9.5/lib/libxml2.so.2 (0x00007ffd5daf7000)
	libm.so.6 => /lib64/libm.so.6 (0x00007ffd5d867000)
	libc.so.6 => /lib64/libc.so.6 (0x00007ffd5d4d2000)
	libgeos-3.6.1.so => /usr/local/geos/lib/libgeos-3.6.1.so (0x00007ffd5d118000)
	libstdc++.so.6 => /usr/lib64/libstdc++.so.6 (0x00007ffd5ce12000)
	libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x00007ffd5cbfb000)
	libpthread.so.0 => /lib64/libpthread.so.0 (0x00007ffd5c9de000)
	libdl.so.2 => /lib64/libdl.so.2 (0x00007ffd5c7da000)
	libz.so.1 => /opt/PostgreSQL/9.5/lib/libz.so.1 (0x00007ffd5c5be000)
	libiconv.so.2 => /opt/PostgreSQL/9.5/lib/libiconv.so.2 (0x00007ffd5c2cf000)
	/lib64/ld-linux-x86-64.so.2 (0x0000003db3600000)

以上错误是由于新安装的插件动态库文件未加载到系统，手工加载如下：

vi /etc/ld.so.conf.d/proj.conf
/usr/local/proj/lib
ldconfig

vi /etc/ld.so.conf.d/geos.conf
/usr/local/geos/lib
ldconfig

vi /etc/ld.so.conf.d/gdal.conf
/usr/local/gdal/lib
ldconfig
ldconfig通常在系统启动时运行，而当用户安装了一个新的动态链接库时，就需要手工运行这个命令。

gis=# create extension postgis_topology;
