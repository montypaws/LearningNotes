#MySQL常用命令详解#
数据库登陆命令：`mysql -uroot -p`。

数据库root用户密码修改：`登陆以后输入set password for 'root'@loclahost'=password('NEW_PASSWORD')`。

数据库退出：`quit;`或者`exit；`（不要忘记输入结尾的分号！）。

创建数据库并设置数据库的字符集：**`create database DATABASE_NAME default character set CHARACTER_NAME`**。

创建表并设置数据库的字符集：`create table TABLE_NAME (id int(NUM),name varchar(LENGTH)) default character set CHARACTER_NAME`。

修改表中字段的长度：`alter table TABLENAME modify column VARIABLE varchar(NEWLENGTH);`。

增加字段：`alter table TABLENAME add NEW_FIELD type；`。

删除字段：`alter table TABLENAME drop column NEW_FIELD；`。

向数据库中添加数据：`insert into TABLENAME (FIELD1,FEILD2,...) values('VALUE1','VALUE2',...);`。

向数据库中添加多条数据：`insert into TABLENAME (FIELD1,FEILD2,...) values('VALUE1','VALUE2',...)，（'VALUE1','VALUE2',...），（'VALUE1','VALUE2',...）...;`。

使用set向数据库中添加数据： `insert into TABLENAME set FIELD1='VALUE1',FEILD2='VALUE2' `。此语法非标准，谨慎使用。

修改数据库中的数据：`update TABLENAME SET FIELD='VALUE' WHERE FILED='VALUE';`。

删除数据库中的数据：`delete from TABLENAME WHERE FILED='VALUE';`。

清空整个表：`delete from TABLENAME;`。

查看数据库的字符集：`show create database DATABASE_NAME；`

查看表的字符集：`show create table TABLE_NAME；`

查看字符的字符集：`show full column from TABLE_NAME；`

修改数据库的字符集：`alter database DATABASENAME default character set CHARACTERNAME；`

修改表的字符集：`alter table TABLENAME default character set CHARACTERNAME；`

修改表和所有字符的字符集：`alter table TABLENAME convert to  character set CHARACTERNAME；`

修改字符的字符集：`alter table TABLENAME change FIELDNAME FIELDNAME character set CHARACTERNAME；`