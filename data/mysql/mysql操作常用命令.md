# mysql操作常用命令

## mysql连接

```sql
# mysql指定远程链接
mysql -h 主机地址 -P端口 -u 用户名 -p 密码
```



## 基本操作

### 简单库操作

```sql
use 数据库名;  # 切换数据库
show databases;  # 展示数据库
```



### 表操作

创建表

```sql
create table if not exists 表名 (
  列名1   数据类型   [列的属性],
  列名2   数据类型   [列的属性],
  ...
  列名n   数据类型   [列的属性]
) comment '表的注释信息';
```

删除表: `drop table if exists 表1, ..., 表n;`

查看表结构:

```sql
describe 表名;
desc 表名;
explain 表名;
show columns from 表名;
show fields from 表名;
show create table 表名;
```

修改表

改表名: `rename table 旧表名1 to 新表名1, ... 旧表名n to 新表名n;`



### 列操作

增加列:

```sql
alter table 表名 add column 列名 数据类型 [列的属性];
alter table 表名 add column 列名 列的类型 [列的属性] first;  # 增加列到第一列
alter table 表名 add column 列名 列的类型 [列的属性] after 指定列名;  # 添加到指定列的后边
```

删除列:

```sql
alter table 表名 drop column 列名;
```

修改列:

```sql
alter table 表名 modify 列名 新数据类型 [新属性];  # 仅修改列属性
alter table 表名 modify 列名 列的类型 列的属性 first;  # 修改列属性并移动到第一列
alter table 表名 modify 列名 列的类型 列的属性 after 指定列名;  # 修改列属性并移动到指定列后面
alter table 表名 change 旧列名 新列名 新数据类型 [新属性];  # 修改列名称及属性
alter table 表名 操作1, 操作2, ..., 操作n;  # 多个修改操作
```

索引操作

```sql
alter table table_name add primary key (column);  # 增加primary key(主键索引):
alter table table_name add unique (column);  # 增加unique(唯一索引):
alter table table_name add index index_name (column);  # 增加index(普通索引):
alter table table_name add fulltext (column);  # 增加fulltext(全文索引):
alter table table_name add index index_name (column1,column2,column3);  # 增加多字段索引:
drop index index_name  on table_name;  # 删除索引:
```



## 复杂操作

创建临时表并把已有的表数据添加到临时表

`create table 临时表 select 列名1, 列名2, ..., 列名n from 查询表;`

查出一张表的字段插入临时表

```sql
insert into 临时表 (列名1, 列名2, ..., 列名n) values (列名a,列名b, ... (select 列名n from t_users));

或

insert into 临时表 (列名1, 列名2, ..., 列名n) (select select 列名1, 列名2, ..., 列名n from 查询表);

insert into 临时表 (列名1, 列名2, ..., 列名n) select select 列名1, 列名2, ..., 列名n from 查询表;
```

### 关联操作

关联更改

`update ... lefe join ...`

关联删除

`delete ... left join ...`

### 替换指定字符

`REPLACE(str,from_str,to_str)`

eg: update cg_mat set pgh_code = replace(pgh_code,"需求","pgh");





## 数据库

### 创建库和用户

创建数据库

```sql
create database `数据库` default character set utf8mb4 collate utf8mb4_unicode_ci;  # 创建 utf8mb4 数据库
create database `数据库` default character set utf8 collate utf8_general_ci;  # 创建 utf8 数据库
```

创建用户

```sql
create user 'userName'@'%' identified by 'password';  # 创建用户
grant all privileges on [数据库].* to 'userName';  # 用户授权数据库 
grant select,insert,update,delete,create,drop on [数据库].* to 'userName';  # 用户授权数据库 
```



### 用户授权

```sql
# 新建用户并指定数据库授所有权限
grant all privileges on `数据库`.* to 'userName'@'%' identified by 'password' with grant option;

# 新建用户并授所有权限
grant all privileges on *.* to 'userName'@'%' identified by 'password' with grant option;
# with grant option; 表示该用户可以将自己拥有的权限授权给别人
# % 表示所有ip都能访问,这里可以指定ip,也可以写localhost

grant select on 'dbname'.* to 'userName'@'%' identified by 'password';  # 新建只读用户 

flush privileges;  # 刷新服务

# 给root授予所有权,包括远程访问
grant all privileges on *.* to 'root'@'%' identified by 'password' with grant option;  
```



### 取消权限和用户

```sql
revoke all on *.* from userName;  # 取消用户所有数据库(表)的所有权限
delete from mysql.user where user='userName';  # 删除用户
drop database if exists [数据库];  # 删除数据库  
drop user userName@localhost;  # 删除账户
select host,user from user;  # 查询数据库账号和权限
```



## mysql导入导出数据库

```sql
mysqldump -u用户名 -p密码 数据库名 > 名称.sql  # 导出数据量
mysql -uroot -p123456 mydb < /data/test.sql;  # 导入sql文件, 不用登录数据库操作
source  /data/test.sql;  # 导入sql文件, 登录数据库操作
```





## 存储程序

自定义变量

```
set @a = 1;  # 设置自定义变量并赋值
set @a = (select m1 from t1 limit 1);  # 设置自定义变量并赋值
select n1 from t1 limit 1 into @b;  # 将查询的结果赋值给一个变量
select m1, n1 from t1 limit 1 into @a, @b;  # 查询结果是有多个列的值，把这几个列的值分别赋值到不同的变量中
set @b = @a;  # 把一个变量的值赋值给另一个变量

select @a;  # 查看自定义变量的值
```

语句结束分隔符

默认语句结束分隔符:  `;`、`\g`或者`\G`

自定义语句结束分隔符:  `delimiter 分隔符` ,  eg:  `delimiter $`



### 存储函数

定义

```
create function 存储函数名称([参数列表])
returns 返回值类型
begin
    函数体内容(一条或多条语句)
end
```

eg

```
create function get_user(a varchar(100))
returns varchar
begin
    return (select group_concat() from user where name = a);
end;
```

调用

```
select get_user('darkoud');
```



其他操作

```
show create function 函数名  # 查看
drop function 函数名  # 删除
```



### 函数体定义

在函数体中定义局部变量

```
declare 变量名1, 变量名2, ... 数据类型 [default 默认值]; # 在函数体中定义局部变量,未赋值则默认值是null,declare语句必须放到其他语句的前边
```

eg: declare c int default 1;  # 声明一个名称为c的int类型的默认值为1的局部变量

在函数体中使用自定义变量

```
set @abc = 10;
```



存储函数的参数

```
参数名 数据类型 # 定义存储函数时, 可以指定多个参数, 用逗号分隔
```

> - 每个参数都要指定对应的数据类型
> - 参数名不能和函数体语句中的其他变量名, 查询语句列名冲突
> - 函数参数不可以指定默认值, 在调用函数的时必须显式的指定所有的参数, 并且参数类型一定要匹配



### 判断语句

```
if 表达式 then
    处理语句列表
[elseif 表达式 then
    处理语句列表]  # 处理语句列表中可以包含多条语句，每条语句以分号;结尾
... # 这里可以有多个elseif语句
[else
    处理语句列表]
end if;
```



### 循环语句

`WHILE`循环语句

```
while 表达式 do
    处理语句列表
end while;
# 先判断表达式的值，再执行处理语句
```



`REPEAT`循环语句

```
repeat
    处理语句列表
until 表达式 end repeat;
# 先执行处理语句，再判断表达式的值，至少执行一次处理语句
```



`LOOP`循环语句

```
loop
    处理语句列表
end loop;
```

> `LOOP`循环终止条件写在处理语句里，终止方式有俩种：
>
> 1. 使用`return`语句返回
     >
     >    ```
>    create function sum_all(n int unsigned)
>    returns int
>    begin
>        declare result int default 0;
>        declare i int default 1;
>        loop
>            if i > n then
>                return result;
>            end if;
>            set result = result + i;
>            set i = i + 1;
>        end loop;
>    end
>    ```
>
> 2. 使用`LEAVE`语句
     >
     >    ```
>    create function sum_all(n int unsigned)
>    returns int
>    begin
>        declare result int default 0;
>        declare i int default 1;
>        # 在loop语句前加了一个flag:，相当于为这个循环打了一个名叫flag的标记
>        flag:loop
>            if i > n then
>             	# 使用leave flag语句来结束flag这个标记所代表的循环
>                leave flag;
>            end if;
>            set result = result + i;
>            set i = i + 1;
>        end loop flag;
>        return result;
>    end
>    ```



### 存储过程

创建

```
create procedure 存储过程名称([参数列表])
begin
    需要执行的语句
end
```

与`存储函数`最直观的不同点就是, `存储过程`的定义不需要声明`返回值类型`

调用

```
call 存储过程([参数列表]); #存储过程在执行中产生的所有结果集，全部将会被显示到客户端
```

> 只有查询语句才会产生结果集



查看与删除

```
show procedure status [like 需要匹配的存储过程名称] # 查看当前数据库中有哪些存储过程
show create procedure 存储过程名称  # 查看某个存储过程具体定义语句

drop procedure 存储过程名称  # 删除指定存储过程
```

> `存储函数`中使用到的各种语句, 包括变量的使用, 判断, 循环结构都可以被用在`存储过程`中



参数前缀

比`存储函数`强大的一点是, `存储过程`在定义参数的时候可以选择将不同参数定义为不同的类型

```
参数类型[IN | OUT | INOUT] 参数名 数据类型
```

| 前缀（参数类型） | 实际参数是否必须是变量 | 描述                                                         |
| :--------------: | ---------------------- | ------------------------------------------------------------ |
|        IN        | 否                     | 仅用于调用者向存储过程传递数据，缺省值                       |
|       OUT        | 是                     | 仅用于把存储过程运行过程中产生的数据赋值给OUT参数，可供外部使用 |
|      INOUT       | 是                     | 可以用于调用者向存储过程传递数据，也可以用于存放存储过程中产生的数据以供调用者使用 |

eg

```
create procedure get_score_data(
        out max_score double,
        out min_score double,
    	out avg_score double,
    	s varchar(100)
    )
begin
    select max(score), min(score), avg(score) from student_score where subject = s into max_score, min_score, avg_score;
end;
```



#### 存储过程和存储函数的不同点

- 存储函数在定义时需要显式用`RETURNS`语句标明返回的数据类型, 而且在函数体中必须使用`RETURN`语句来显式指定返回的值, 存储过程不需要
- 存储函数只支持`IN`参数, 而存储过程支持`IN`参数, `OUT`参数, 和`INOUT`参数
- 存储函数只能返回一个值, 而存储过程可以通过设置多个`OUT`参数或者`INOUT`参数来返回多个结果
- 存储函数执行过程中产生的结果集并不会被显示到客户端, 而存储过程执行过程中产生的结果集会被显示到客户端
- 存储函数直接在表达式中调用, 而存储过程只能通过`CALL`语句来显式调用