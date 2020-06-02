[TOC]

# Oracle数据库

## 一、数据库

### 1、创建数据库

     create database databasename

### 2、删除数据库

     drop database dbname

### 3、备份数据库

完全备份
     exp demo/demo@orcl buffer=1024 file=d：\back.dmp full=y

     demo：用户名、密码
    
     buffer: 缓存大小
    
     file: 具体的备份文件地址
    
     full: 是否导出全部文件
    
     ignore: 忽略错误，如果表已经存在，则也是覆盖

将数据库中system用户与sys用户的表导出
     exp demo/demo@orcl file=d:\backup\1.dmp owner=(system,sys)

导出指定的表
     exp demo/demo@orcl file=d:\backup2.dmp tables=(teachers,students)

按过滤条件，导出
     exp demo/demo@orcl file=d：\back.dmp tables=（table1） query=\" where filed1 like 'fg%'\"

     导出时可以进行压缩；命令后面 加上 compress=y ；如果需要日志，后面： log=d:\log.txt

备份远程服务器的数据库
     exp 用户名/密码@远程的IP:端口/实例 file=存放的位置:\文件名称.dmp full=y

### 4、数据库还原

     打开cmd直接执行如下命令，不用再登陆sqlplus。

完整还原
     imp demo/demo@orcl file=d:\back.dmp full=y ignore=y log=D:\implog.txt

     指定log很重要，便于分析错误进行补救。

导入指定表
     imp demo/demo@orcl file=d:\backup2.dmp tables=(teachers,students)

还原到远程服务器
     imp 用户名/密码@远程的IP:端口/实例 file=存放的位置:\文件名称.dmp full=y

## 实例的创建



## 表空间

### 1、表空间概述与默认表空间

  **重点内容表空间有三种：小文件表空间（默认）、大文件表空间、临时表空间。**

 **默认表空间表示**<font color="red">，当建表，建索引等的时候，如果不指定表放在哪里，那么自动放在该用户的默认表空间（创建一个用户的时候需要指定他的默认表空间）。这就印证了：一个用户可以使用一个或多个表空间，一个表空间也可以供多个用户使用。</font>

Oracle11g数据库存在6个默认表空间：EXAMPLE、SYSAUX、SYSTEM、TEMP、UNDOTBS1、USERS。其中：

```mysql
  EXAMPLE表空间：用于安装Oracle 11g数据库使用示例数据库。
  SYSAUX表空间：作为EXAMPLE表空间的辅助表空间。
  SYSTEM表空间：用来存储SYS用户的表、视图、、存储过程等数据库对象。
  TEMP表空间：临时表空间，用户SQL语句处理的表和索引信息。
  UNDOTBS1表空间：用于存储撤销信息。
  USERS表空间：存储数据库用户创建的数据库对象。
```
### 2、查看所有用户的默认表空间

   通过数据库字典DBA_TABLESPACES查看所有默认表空间。

```mysql
select * from dba_tablespaces
```

### 3、查看指定用户默认表空间

   通过数据库字典DBA_USERS查看指定用户的默认表空间。

```
select * from dba_users
```

### 表空间管理

   **表空间管理的四种情况：创建、修改、删除、扩容。**

### 1、创建表空间

​      创建表空间的一般语法：
​      （语法符号解析：中括号[]表示可有可无，竖线 | 表示或者, 大括号{ }表示必须要有的）

```mysql
CREATE TABLESPACE tablespace_name
DATAFILE 'filename'  SIZE size 
[AUTOEXTEND  [OFF|ON NEXT size] ]  
[MAXSIZE size]
[PERMANENT|TEMPORARY]
[EXTENT MANAGEMENT 
	[DICTIONARY|LOCAL
		[AUTOALLLOCATE|UNIFORM.[SIZE integer[K|M]]]
    ]
 ]
```

**【语法说明】**
          **TABLESPACE** ：指定要创建表空间的名称
          **DATAFILE**：指定表空间的数据文件名称，还要指定文件的路径。
         **AUTOEXTEND** ：指定表空间数据文件的自动扩展方式。ON表示自动扩展，OFF表示非自动扩展。如果使用自动扩展应该在NEXT关键字后指定具体大小。
          **MAXSIZE** : 指定数据文件自动扩展方式时的最大值。
        **PERMANENT|TEMPORARY** ：指定表空间的类型，PERMANENT表示永久表空间；TEMPORARY表示临时表空间。
          **EXTENT MANAGEMENT** ：指定表空间的管理方式，DICTIONARY表示字典管理方式，LOCAL表示本地管理方式。（默认使用本地管理方式，也是推荐使用的）

示例：

```mysql
CREATE TABLESPACE testone
DATAFILE 'testone.dbf'  SIZE 10M 
AUTOEXTEND  ON NEXT 128k
MAXSIZE 2048M
```

### 2、重命名表空间

​      重命名表空间语法：

```mysql
ALTER TABLESPACE old_name to new_name
```

**【说明】**
*不是所有的表空间的都可以重命名，YSYTEM、SYSAUX就不能，处于OFFLINE状态的表空间也不能重命名。
*要重命名表空间的前提是，表空间存在，可以在数据字典中查看表空间的名字在进行操作。

示例：

```
ALTER TABLESPACE testone to testonenew
```

### 3、设置大文件表空间

​      **设置大文件表空间是在正常表空间不足够使用的时候才创建。**
​      设置大文件表空间的语法：

```plsql
CREATE BIGFILE TABLESPACE tablespace_name  DATAFILE filename SIZE size
```

**【语法说明】**
      **tablespace** :表空间
      **filename** ：设置大文件名
      **SIZE**：设置大文件的大小

示例：

```plsql
CREATE BIGFILE TABLESPACE testbigfile DATAFILE 'bigfiletest.dbf' SIZE 2G
```

### 4、删除表空间

​      **删除表空间的时候可以选择将表空间的文件和完整性一起删除。**
​      删除表空间的语法：

```plsql
DROP TABLESPACE tablespace_name [INCLUDING CONTENTS]  [CASCADE CONSTRAINTS]
```

**【语法说明】**

1. **INCLUDING CONTENTS**: 包含数据文件
2. **CASCADE CONSTRAINTS** : 包含表空间完整性

示例：

DROP TABLESPACE testone INCLUDING CONTENTS 


### 5 、扩容表空间

1. 给表空间增加数据文件

   ```plsql
   ALTER TABLESPACE app_data ADD DATAFILE  
   'D:\ORACLE\PRODUCT\10.2.0\ORADATA\EDWTEST\APP03.DBF' SIZE 50M;  
   ```

2. 新增数据文件，并且允许数据文件自动增长

   ```plsql
   ALTER TABLESPACE app_data ADD DATAFILE
   'D:\ORACLE\PRODUCT\10.2.0\ORADATA\EDWTEST\APP04.DBF' SIZE 50M
   AUTOEXTEND ON NEXT 5M MAXSIZE 100M;
   ```

3. 允许已存在的数据文件自动增长

   ```plsql
   ALTER DATABASE DATAFILE 'D:\ORACLE\PRODUCT\10.2.0\ORADATA\EDWTEST\APP03.DBF'  
   AUTOEXTEND ON NEXT 5M MAXSIZE 100M;  
   ```

4. 手工改变已存在数据文件的大小

   ```plsql
   ALTER DATABASE DATAFILE 'D:\ORACLE\PRODUCT\10.2.0\ORADATA\EDWTEST\APP02.DBF'  
   RESIZE 100M;
   ```

   

   ###  临时表空间管理

   ​      Oracle临时表空间主要用来做查询和存放一些缓冲区数据。临时表空间消耗的主要原因是需要对查询的中间结果进行排序。
   ​      重启数据库可以释放临时表空间，如果不能重启实例，而一直保持问题sql语句的执行，temp表空间会一直增长。直到耗尽硬盘空间。
   ​      存放临时数据（一般是排序结果），数据库存储数据，当内存不够时存入临时表空间，当执行完数据库操作后，自动清空。

------

## 用户

### 1、创建用户

```plsql
CREATE USER OT IDENTIFIED BY Orcl1234;
说明：CREATE USER 表名 IDENTIFIED BY 密码;
```

### 2、查询当前用户下的表

```mysql
 SELECT table_name FROM user_tables ORDER BY Table_name
 说明：从user_tables表中选择了table_name列中的值，并按字母顺序排列了表名
```



## 权限



## 表操作

### 1、创建表

```plsql
 create table tabname(col1 type1 [not null] [primary key],col2 type2 [not null],..)
 根据已有的表创建新表：
 A：select * into table_new from table_old (使用旧表创建新表)
 B：create table tab_new as select col1,col2… from tab_old definition only<仅适用于Oracle>
```

### 2、删除表

     drop table tabname

### 3、重命名表

     说明：alter table 表名 rename to 新表名
        eg：alter table tablename rename to newtablename

### 4、增加字段

     说明：alter table 表名 add (字段名 字段类型 默认值 是否为空);
        例：alter table tablename add (ID int);
       eg：alter table tablename add (ID varchar2(30) default '空' not null);

### 5、修改字段

     说明：alter table 表名 modify (字段名 字段类型 默认值 是否为空);
        eg：alter table tablename modify (ID number(4));

### 6、重名字段

     说明：alter table 表名 rename column 列名 to 新列名 （其中：column是关键字）
        eg：alter table tablename rename column ID to newID;

### 7、删除字段

     说明：alter table 表名 drop column 字段名;
        eg：alter table tablename drop column ID;

### 8、添加主键

     alter table tabname add primary key(col)

### 9、	删除主键

     alter table tabname drop primary key(col)

### 10、创建索引

     create [unique] index idxname on tabname(col….)

### 11、删除索引

     drop index idxname
    
     注：索引是不可更改的，想更改必须删除重新建。

### 12、创建视图

     create view viewname as select statement

### 13、删除视图

     drop view viewname

## 表数据操作



## PL/SQL编程



### 1、变量定义

### 2、流程控制

2.1 if判断

2.2循环



### 3、游标