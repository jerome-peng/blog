#小版本升级数据库后对现有的database需要进行如下操作

## 1 修改postgresql.conf添加allow_system_table_mods = true，然后重启数据库

## 2 每个数据库使用超级用户执行如下命令

SET search_path = pg_catalog;
CREATE OR REPLACE VIEW pg_user_mappings AS
    SELECT
        U.oid       AS umid,
        S.oid       AS srvid,
        S.srvname   AS srvname,
        U.umuser    AS umuser,
        CASE WHEN U.umuser = 0 THEN
            'public'
        ELSE
            A.rolname
        END AS usename,
        CASE WHEN (U.umuser <> 0 AND A.rolname = current_user)
                    OR (U.umuser = 0 AND pg_has_role(S.srvowner, 'USAGE'))
                    OR (SELECT rolsuper FROM pg_authid WHERE rolname = current_user)
                    THEN U.umoptions
                 ELSE NULL END AS umoptions
    FROM pg_user_mapping U
         LEFT JOIN pg_authid A ON (A.oid = U.umuser) JOIN
        pg_foreign_server S ON (U.umserver = S.oid);
        
## 3 修改template0 和 template1

ALTER DATABASE template0 WITH ALLOW_CONNECTIONS true;

参考步骤2

ALTER DATABASE template0 WITH ALLOW_CONNECTIONS false;

## 4移除postgresql.conf里allow_system_table_mods，重启数据库

参考https://www.postgresql.org/docs/9.5/static/release-9-5-7.html
