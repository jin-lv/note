# oracle tablespace,user等
## linux:
1.  su - oracle
2.  用sqlplus:  sqlplus / as sysdba

## 创建表空间和临时表空间
###### linux :
```sql
create tablespace A datafile '/home/data/A.dbf' size 1000m autoextend on next 50m maxsize 20480m ;
```
```sql
create temporary tablespace A_TEMP tempfile  '/home/data/A_TEMP.dbf' size 1000m autoextend on next 500m maxsize 20480m ;
```
###### window:
```sql
create tablespace A_DATA datafile 'D:\app\ora\A_DATA.ora' size 1000m autoextend on next 50m maxsize 20480m ;
```
```sql
create temporary tablespace A_TEMP tempfile  'D:\app\ora\A_TEMP.ora' size 1000m autoextend on next 500m maxsize 20480m ;
```

###### 删除表空间
```sql
drop tablespace thf_ws  including contents and datafiles cascade constraints;
```

###### 查询表所属表空间信息：
```sql
select table_name ,tablespace_name from dba_tables;
```

###### 只查询表空间
```sql
select * from dba_tablespaces
select TABLESPACE_NAME from dba_tablespaces;
select TABLESPACE_NAME from tablespaces;
```
###### 查看数据库中当前的sid name：

```sql
select INSTANCE_NAME from v$instance;
```


## 用户
###### 创建用户
```sql
create user username identified by password default tablespace A_data temporary tablespace A_temp;
```

###### 查询oracle用户
```sql
select username from dba_users order  by username;
```

###### 重置用户密码：
```sql
alter user username identified by 11111111;
```

###### 删除用户
```sql
drop user username cascade
```

###### 给用户授权
```sql
grant connect,resource,dba to username;
```


## directory
###### 创建directory，导入导出的时候用
```sql
create directory dump_dir as '/u01/app/dbback';
```

###### 查询directory
```sql
select * from dba_directories;
```


## 导入
> remap_schema当你从A用户导出的数据，想要导入到B用户中去，就使用这个：remap_schema=A:B

> remap_tablespace 与上面类似，数据库对象本来存在于tbs_a表空间，现在你不想放那儿了，想换到tbs_b，就用这个remap_tablespace=tbs_a:tbs_b 结果是所有tbs_a中的对象都会建在tbs_b表空间中。这样做的前提是目标用户B和目标表空间tbs_b存在


```sql
impdp username/password@orcl
directory=backup_dump
dumpfile=20180204.dmp
remap_schema=thf:thf,thfdw:thfdw
logfile=A_20180204.log parallel=2
table_exists_action=replace
remap_tablespace = A : A ;
```
```sql
impdp username/password@orcl
directory=data_pump_dir
dumpfile=20151207.dmp
logfile=20151207.log  
remap_schema=thf:thf
table_exists_action = replace
remap_tablespace = A : A ;
```
```sql
impdp username/password@orcl
directory=dir
dumpfile=20180309.dmp
logfile=impdp_fxg6_%date:~0,4%%date:~5,2%%date:~8,2%_log.txt
remap_schema=fxdm:fxdm,fxAgp:fxAgp,fxdw:fxdw,fxkettle:fxkettle,fxods:fxods
remap_tablespace=fxbase001:Abase001
parallel=4
```

## 导出
```sql
Expdp username/password  
directory=dump_dir
dumpfile = ys_Agprm.dmp
logfile = xxx.log
schemas= username
```

###### 导入多个用户，前提是用户名不一致
```sql
impdp 用户名/密码@203orcl
directory=hmn6
dumpfile=A20150408_2.dmp
remap_schema=hncbAgp:hmAgp,hncbdw:hmdw,hncbdm:hmdm,hncbkettle:hmkettle
logfile=imp_hm20150408.log
parallel=2
table_exists_action=replace
```

###### 分多文件导出
```sql
expdp username/password dumpfile=hm%U.dmp logfile=hm.log parallel=2
```
 > %U可以指定导出线程数 个文件，此处parallel=2,导出的文件是2个

###### 导入多个文件，增加逗号即可,前提是用户名一致
```sql
impdp hmn6/hmn6@203orcl
directory=hmn6
dumpfile=hmcw1.dmp,hmcw2.dmp
parallel=2
remap_schema=hmcw:hmn6
table_exists_action=replace
```

###### 导出多个用户
```sql
impdp 用户名/密码@203orcl
directory=HMN6
dumpfile=A20150408_2.dmp  
schemas=a,b,c
logfile=imp_hm20150408.log
parallel=2
table_exists_action=replace
```

###### 导入多个用户，前提是用户名不一致
```sql
impdp 用户名/密码@203orcl
directory=HMN6
dumpfile=A20150408_2.dmp  
remap_schema=hncbAgp:hmAgp,hncbdw:hmdw,hncbdm:hmdm,hncbkettle:hmkettle
logfile=imp_hm20150408.log
parallel=2
table_exists_action=replace
```

###### 加密导出、导入
```sql
导出：expdp 用户名/密码
directory = dir
dumpfile = secooler.dmp
logfile = secooler.log
encryption = data_only
encryption_password = my_passwd
```
```sql
导入：impdp 用户名/密码
directory = dir
dumpfile = secooler.dmp
logfile = secooler_impdp.log
encryption_password = my_passwd
```

###### 远程导出到本地
> network_link为连接远程的databaselink

```sql
expdp user/pwd@orcl directory=dd network_link=dblink dumpfile=fileName.dmp
```




###### 查询死锁
```sql
SELECT username, lockwait, status, machine, program
FROM v$session
WHERE sid IN (
	SELECT session_id
	FROM v$locked_object
);
```

###### 创建dblink
```sql
drop database link LINK;
create database link LINK
  connect to dw IDENTIFIED dw
  using '(DESCRIPTION =
           (ADDRESS_LIST =
               (ADDRESS =
                  (PROTOCOL = TCP)
                  (HOST = 172.0.0.1)
                  (PORT = 1521)) )
               (CONNECT_DATA =
                   (SERVICE_NAME = thfdb)
          ))';
```
###### 查询所有表名：
```sql
select t.table_name from user_tables t;
```
###### 查询所有字段名：
```sql
select t.column_name
from user_col_comments t;
```
###### 查询指定表的所有字段名：
```sql
SELECT t.column_name
FROM user_col_comments t
WHERE t.table_name = 'BIZ_DICT_XB';
```
###### 查询指定表的所有字段名和字段说明：
```sql
SELECT t.column_name, t.column_name
FROM user_col_comments t
WHERE t.table_name = 'BIZ_DICT_XB';
```
###### 查询所有表的表名和表说明：
```sql
SELECT t.table_name, f.comments
FROM user_tables t
	INNER JOIN user_tab_comments f ON t.table_name = f.table_name ;
```
###### 查询模糊表名的表名和表说明：
```sql
SELECT t.table_name
FROM user_tables t
WHERE t.table_name LIKE 'BIZ_DICT%';
```
```sql
SELECT t.table_name, f.comments
FROM user_tables t
	INNER JOIN user_tab_comments f ON t.table_name = f.table_name
WHERE t.table_name LIKE 'BIZ_DICT%';
```

###### 查询表的数据条数、表名、中文表名
```sql
SELECT a.num_rows, a.TABLE_NAME, b.COMMENTS
FROM user_tables a, user_tab_comments b
WHERE a.TABLE_NAME = b.TABLE_NAME
ORDER BY TABLE_NAME
```
###### 查询表空间
```sql
select b.file_id 文件ID号,
       b.tablespace_name 表空间名,
       b.bytes / 1024 / 1024 / 1024 || 'G' 字节数,
       (b.bytes - sum(nvl(a.bytes, 0))) / 1024 / 1024 / 1024 || 'G' 已使用,
       sum(nvl(a.bytes, 0)) / 1024 / 1024 || 'M' 剩余空间,
       100 - sum(nvl(a.bytes, 0)) / (b.bytes) * 100 占用百分比
  from dba_free_space a, dba_data_files b
 where a.file_id = b.file_id
 group by b.tablespace_name, b.file_id, b.bytes
 order by b.file_id;
 ```
