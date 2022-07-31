# Mysql 基础篇

> 问题：接单被问道做 存储过程里面字段的血缘进行解析。 1.存储过程？ 2. 字段的血缘关系？字段的映射？ 3. MyBatis



## 安装

略。。。

安装完成之后因为默认root密码为空，所以要创建密码

```sql
use mysql;
alter user 'root'@'localhost' identified with mysql_native_password by '密码';
```

连接测试

```shell
mysql -u root -p 
```



## 启动、关闭

```shell
# 启动
# /usr/bin/mysqld_safe &  
systemctl start mysql
# 关闭
/usr/bin/mysqladmin -u root -p shutdown
或者
systemctl stop mysql
```

## Msql管理

### 用户设置

MySql8有新的安全要求，不能像之前的版本那样一次性创建用户并授权需要先创建用户，再进行授权操作

``` 
# 连接 mysql
# mysql -u root -p
Enter password:*********
mysql> use mysql;
Database changed
# 创建新用户
mysql> create user 'username'@'host' identified by 'password';
# 用户授权 *.* 第一个标识数据库，第二个表示表 %为该用户登录的域名
mysql> grant all privileges on *.* to 'username'@'%' with grant option; 

# 授权之后刷新权限
mysql> flush privileges; 

# 撤销授权
#收回权限(不包含赋权权限)
REVOKE ALL PRIVILEGES ON *.* FROM user_name;
REVOKE ALL PRIVILEGES ON user_name.* FROM user_name;
#收回赋权权限
REVOKE GRANT OPTION ON *.* FROM user_name;

#操作完后重新刷新权限
flush privileges;

```



insert 用户时使用PASSWORD()对密码进行加密处理。

**注意：**在 MySQL5.7 中 user 表的 password 已换成了**authentication_string**。

**注意：**password() 加密函数已经在 8.0.11 中移除了，可以使用 MD5() 函数代替。

**注意：**在注意需要执行 **FLUSH PRIVILEGES** 语句。 这个命令执行后会重新载入授权表。

如果你不使用该命令，你就无法使用新创建的用户来连接mysql服务器，除非你重启mysql服务器。



另外一种添加用户的方法为通过SQL的 GRANT 命令。

```shell
mysql> GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP
    -> ON TUTORIALS.*
    -> TO 'zara'@'localhost'
    -> IDENTIFIED BY 'zara123';
```



### /etc/my.cnf 默认配置

```ini
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

[mysql.server]
user=mysql
basedir=/var/lib

[safe_mysqld]
err-log=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```

### 管理mysql的命令

- USE 数据库名

  ```mysql
  use RUNOOB;
  ```

  

- SHOW DATABASES：

  ```mysql
  SHOW DATABASES;
  ```

  

- SHOW TABLES:

  显示指定数据库的所有表，使用该命令前需要使用 use 命令来选择要操作的数据库。

  ```mysql
  SHOW TABLES;
  ```

- SHOW COLUMNS FROM 数据表:

  显示数据表的属性，属性类型，主键信息 ，是否为 NULL，默认值等其他信息。

  ```mysql
  SHOW COLUMNS FROM runoob_tbl;
  ```

- SHOW INDEX FROM 数据表：?

  显示数据表的详细索引信息，包括PRIMARY KEY（主键）。

  ```mysql
  SHOW INDEX FROM runoob_tbl;
  ```

- SHOW TABLE STATUS [FROM db_name] [LIKE 'pattern'] \G:

  该命令将输出Mysql数据库管理系统的性能及统计信息。

  ```mysql
  SHOW TABLE STATUS from RUNOOB LIKE 'runoob%';   # 表名以 runoob开头的表的信息
  SHOW TABLE STATUS from RUNOOB LIKE 'runoob%' \G;  # 加上 \G,查询结果按列打印
  
  ```

## Python 使用mysql

python下使用pymysql用于连接mysql数据库和做响应的操作

[参考](https://www.runoob.com/python3/python3-mysql.html)

- 安装

  ```shell
  pip3 install pymysql
  ```

- 测试

  ```python
  # 一下关于python执行，默认导入pymysql
  import pymsql
  ```

  

- 连接数据库

  ```python
  #!/usr/bin/python3
   
  import pymysql
   
  # 打开数据库连接
  db = pymysql.connect(host='localhost',
                       user='testuser',
                       password='test123',
                       database='TESTDB')
   
  # 使用 cursor() 方法创建一个游标对象 cursor
  cursor = db.cursor()
   
  # 使用 execute()  方法执行 SQL 查询 
  cursor.execute("SELECT VERSION()")
   
  # 使用 fetchone() 方法获取单条数据.
  data = cursor.fetchone()
   
  print ("Database version : %s " % data)
   
  # 关闭数据库连接
  db.close()
  ```

- 创建数据库表

  ```py
  #!/usr/bin/python3
   
  import pymysql
   
  # 打开数据库连接
  db = pymysql.connect(host='localhost',
                       user='testuser',
                       password='test123',
                       database='TESTDB')
   
  # 使用 cursor() 方法创建一个游标对象 cursor
  cursor = db.cursor()
   
  # 使用 execute() 方法执行 SQL，如果表存在则删除
  cursor.execute("DROP TABLE IF EXISTS EMPLOYEE")
   
  # 使用预处理语句创建表
  sql = """CREATE TABLE EMPLOYEE (
           FIRST_NAME  CHAR(20) NOT NULL,
           LAST_NAME  CHAR(20),
           AGE INT,  
           SEX CHAR(1),
           INCOME FLOAT )"""
   
  cursor.execute(sql)
   
  # 关闭数据库连接
  db.close()

- 数据库插入操作

  ```python
  #!/usr/bin/python3
   
  import pymysql
   
  # 打开数据库连接
  db = pymysql.connect(host='localhost',
                       user='testuser',
                       password='test123',
                       database='TESTDB')
   
  # 使用cursor()方法获取操作游标 
  cursor = db.cursor()
   
  # SQL 插入语句
  sql = """INSERT INTO EMPLOYEE(FIRST_NAME,
           LAST_NAME, AGE, SEX, INCOME)
           VALUES ('Mac', 'Mohan', 20, 'M', 2000)"""
  try:
     # 执行sql语句
     cursor.execute(sql)
     # 提交到数据库执行
     db.commit()
  except:
     # 如果发生错误则回滚
     db.rollback()
   
  # 关闭数据库连接
  db.close()
  ```

其他操作基本一致， 修改数据时使用  cursor.execute(sql) 执行sql语句，然后commit提交到数据库执行，如果发生错误使用db.rollback() 进行回滚。

## Msql 连接

```shell
mysql -u root -p
```

python:

```python
mysql_conn = pymysql.connect(host= '127.0.0.1', port= 3306, user= '', password= '', db= '')
# 关闭连接 
mysql_conn.close()
```

## Mysql 创建数据库

sql语句：

```mysql
CREATE DATABASE 数据库名;
```

mysqladmin命令：

```shell
mysqladmin -u root -p create 数据库名
```

## Mysql 删除数据库

sql语句：

```mysql
DROP DATABASE 数据库名;
```

mysqladmin命令：

```shell
mysqladmin -u root -p drop 数据库名
```

## Mysql 选择数据库

sql语句：

```mysql
use 数据库名;
```

## Mysql数据类型

- 数值类型
- 日期和时间类型
- 字符串类型

## MySQL 创建数据表

创建MySQL数据表需要以下信息：

- 表名
- 表字段名
- 定义每个表字段

sql语句：

```sql
create table table_name (column_name column_type);

# 实例
create table if not exists `goods02` (
	`id` int unsigned auto_increment,
  `name` varchar(100) not null,
  `price` varchar(100) not null,
  `date` date,
  primary key (`id`)
)engine=innodb default charset=utf8;

# ``可以省略
```

实例解析：

- 如果你不想字段为 **NULL** 可以设置字段的属性为 **NOT NULL**， 在操作数据库时如果输入该字段的数据为**NULL** ，就会报错。
- AUTO_INCREMENT定义列为自增的属性，一般用于主键，数值会自动加1。
- PRIMARY KEY关键字用于定义列为主键。 您可以使用多列来定义主键，列间以逗号分隔。
- ENGINE 设置存储引擎，CHARSET 设置编码。

## MySQL 删除数据表

Sql:

```sql
drop table table_name;
```

## MySQL 插入数据

Sql:

```sql
insert into table_name (field1, field2, ...fiedN) values (value1, value2, ...valueN);
# 实例
insert into goods (name, price, date) values ('铅笔'，'2$', NOW());
```

## MySQL 查询数据

Sql:

```sql
select column_name, column_name from table_name [where clause] [limit N] [offset m]

# 实例
select * from goods;
select name, price from goods where price <=2 limit 10 offset 1;
```

- 查询语句中你可以使用一个或者多个表，表之间使用逗号(,)分割，并使用WHERE语句来设定查询条件。
- SELECT 命令可以读取一条或者多条记录。
- 你可以使用星号（*）来代替其他字段，SELECT语句会返回表的所有字段数据
- 你可以使用 WHERE 语句来包含任何条件。
- 你可以使用 LIMIT 属性来设定返回的记录数。
- 你可以通过OFFSET指定SELECT语句开始查询的数据偏移量。默认情况下偏移量为0。

## MySQL WHERE 子句

sql:

```sql
select field1, field2,...fieldN from table_name1, table_name2,...table_nameN [where condition1 [and [or] condition2 ...]]
# 实例
select * from goods where binary name="apple"; # binary 代表区分大小写

```

操作符：

```tex
= 
<>,!=
<
>=
<=
```

仅有<> 没有见过，其他和绝大多数语言一致



## MySQL UPDATE 更新

sql:

```sql
update table_name set field1=new-value1, field2=new-value2 [where clause]

# 实例
update goods set price='20$' where name='椅子';
```

## MySQL DELETE 语句

Sql:

```sql
delete from table_name [where clause];
# 实例
delete from goods where id>=10;
```

## MySQL LIKE 子句

Sql:

```sql
select field1, field2,...fieldN from table_name where field1 like condition1 [and [or]] field2='somevalue'

# 实例
select * from goods where name like '%ple'; # 搜索以ple名称结尾的物品
```

## MySQL UNION 操作符

MySQL UNION 操作符用于连接两个以上的 SELECT 语句的结果组合到一个结果集合中。多个 SELECT 语句会删除重复的数据。

SQL：

```sql
select expression1, expression2, ...expressionN from tables [where conditions] union [all | distinct] select expression1, expression2, ...expressionN from tables [where conditions]
```

- **expression1, expression2, ... expression_n**: 要检索的列。
- **tables:** 要检索的数据表。
- **WHERE conditions:** 可选， 检索条件。
- **DISTINCT:** 可选，删除结果集中重复的数据。默认情况下 UNION 操作符已经删除了重复数据，所以 DISTINCT 修饰符对结果没啥影响。
- **ALL:** 可选，返回所有结果集，包含重复数据。

```sql
# 实例
select name from goods union select name from cars order by name;

select name from goods where id>=2 union all select name from cars where id<=10 order by name;
```

## MySQL 排序

SQL：

```sql
select field1, field2,...fieldN from table_name, table_name2... order by field1 [ASC [DESC][默认ASC]]
# 实例
select name from goods, car order by name ASC;
```

​	ASC：升序， DESC：降序

## MySQL GROUP BY 语句

在分组的列上我们可以使用 COUNT, SUM, AVG,等函数。

SQL:

```sql
select column_name, function(column_name) from table_name where column_name operator value group by column_name;

# 实例
# 统计goods中 价格为2的商品数量，然后按价格进行分组
select name, price, count(*) from goods group by price;
```

### 使用 WITH ROLLUP

WITH ROLLUP 可以实现在分组统计数据基础上再进行相同的统计（SUM,AVG,COUNT…）。

Sql:

```sql
select name, sum(signin) as signin_count from employee_tbl GROUP BY name WITH ROLLUP;
```

coalesce 来设置一个可以取代 NUll 的名称

## MySQL 连接的使用

JOIN 按照功能大致分为如下三类：

- **INNER JOIN（内连接,或等值连接）**：获取两个表中字段匹配关系的记录。
- **LEFT JOIN（左连接）：**获取左表所有记录，即使右表没有对应匹配的记录。
- **RIGHT JOIN（右连接）：** 与 LEFT JOIN 相反，用于获取右表所有记录，即使左表没有对应匹配的记录。

```
# inner join (table1 和table2的交集)
select a.field01, a.field02 from table_name01 a inner join table_name02 b on a.field01 = b.field01

select a.field01, b.field02 from table01 a, table02 b where a.field01 = b.field01
```



```sql
# left join (table1 全部，即使 table2中没有和table1的交集)

select a.field01, b.field02 from table01 a left join table02 b on a.field01 = b.field01;
```



```sql
# right join (table2 全部，即使 table1中没有和table2的交集)

select a.field01, b.field02 from table01 a right join table02 b on a.field01 = b.field01;
```

## MySQL NULL 值处理

- **IS NULL:** 当列的值是 NULL,此运算符返回 true。
- **IS NOT NULL:** 当列的值不为 NULL, 运算符返回 true。
- **<=>:** 比较操作符（不同于 = 运算符），当比较的的两个值相等或者都为 NULL 时返回 true。

```sql
# 判定一个值是否为null时，不适用= 或者!=
select * from goods where name is null;
select * from goods where name is not null;
select * from goods where name <=> null;
```

## MySQL 正则表达式

MySQL中使用 REGEXP 操作符来进行正则表达式匹配。

| 模式       | 描述                                                         |
| :--------- | :----------------------------------------------------------- |
| ^          | 匹配输入字符串的开始位置。如果设置了 RegExp 对象的 Multiline 属性，^ 也匹配 '\n' 或 '\r' 之后的位置。 |
| $          | 匹配输入字符串的结束位置。如果设置了RegExp 对象的 Multiline 属性，$ 也匹配 '\n' 或 '\r' 之前的位置。 |
| .          | 匹配除 "\n" 之外的任何单个字符。要匹配包括 '\n' 在内的任何字符，请使用像 '[.\n]' 的模式。 |
| [...]      | 字符集合。匹配所包含的任意一个字符。例如， '[abc]' 可以匹配 "plain" 中的 'a'。 |
| [^...]     | 负值字符集合。匹配未包含的任意字符。例如， '[^abc]' 可以匹配 "plain" 中的'p'。 |
| p1\|p2\|p3 | 匹配 p1 或 p2 或 p3。例如，'z\|food' 能匹配 "z" 或 "food"。'(z\|f)ood' 则匹配 "zood" 或 "food"。 |
| *          | 匹配前面的子表达式零次或多次。例如，zo* 能匹配 "z" 以及 "zoo"。* 等价于{0,}。 |
| +          | 匹配前面的子表达式一次或多次。例如，'zo+' 能匹配 "zo" 以及 "zoo"，但不能匹配 "z"。+ 等价于 {1,}。 |
| {n}        | n 是一个非负整数。匹配确定的 n 次。例如，'o{2}' 不能匹配 "Bob" 中的 'o'，但是能匹配 "food" 中的两个 o。 |
| {n,m}      | m 和 n 均为非负整数，其中n <= m。最少匹配 n 次且最多匹配 m 次。 |

sql：

```sql
# 以 aa开头
select name from goods where name regexp '^aa';
# 以ok结尾
select name from goods where name regexp 'ok$';
# 包含 ss
select name from goods where name regexp 'ss';
# 以 abcd 开头或者ok结尾
select name from goods where name regexp '^[abcd]|ok$'
```

## LeetCode 练习

1. [175.组合两个表（简单）](https://leetcode.cn/problems/combine-two-tables/)

```sql
select a.firstName, a.lastName, b.city, b.state from Person a left join Address b on a.personId = b.personId;
```

2. [176.第二高的薪水](https://leetcode.cn/problems/second-highest-salary/)

```sql
select max(salary) SecondHighestSalary from Employee where salary != (select max(salary) from Employee);
```

3. ##### [177. 第N高的薪水](https://leetcode.cn/problems/nth-highest-salary/)

```sql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
     set n = n -1;
  RETURN (
      # Write your MySQL query statement below.
     
        select distinct salary from employee order by salary desc limit N,1 
      
  );
END
```

4. [超过经理收入的员工](https://leetcode.cn/problems/employees-earning-more-than-their-managers/)

5. [查找重复的电子邮箱](https://leetcode.cn/problems/duplicate-emails/)

   

# Mysql高级篇

[参考小林coding《图解mysql》](https://xiaolincoding.com/mysql/)

## 一条select语句执行的全过程

### 第一步：连接器

也就是连接mysql服务

```shell
mysql -u$username -h$ip -p
```

mysql服务被多少个客户端连接

```sql
show processlist;
```

超时时间

```mysql
show variables like 'wait_timeout';
# 断开连接
kill connection +num;
```

最大连接数

```mysql
show variables like 'max_connections';
```

怎么解决长连接占用内存的问题？

1. 定期断开长链接
2. 客户端重置连接 mysql_reset_connection()

### 第二步：查询缓存

先从缓存找，有直接返回，无往下执行；但更新表是会删除关于此表的所有缓存。

自mysql8.0之后取消的查询缓存，之前版本使用参数query_cache_type 设置成DEMAND。

### 第三步：解析SQL（解析器）

解析器对sql语句进行解析。完成两件事：

1. 词法分析：根据输入指令识别出关键字，例如：select，where，like，inner join等。
2. 语法分析：判断是否是合法语句。

### 第四步：执行SQL

#### 1. 预处理器（prepare）

检查表或者字段是否存在和将select * 中的 * 扩展成表的列。

#### 2. 优化器（optimize）

**优化器主要负责将 SQL 查询语句的执行方案确定下来**，比如在表里面有多个索引的时候，优化器会基于查询成本的考虑，来决定选择使用哪个索引。

#### 3. 执行器（execute）

执行sql语句

## MySQL 索引

### 什么是索引

索引是帮助存储引擎快速获取数据的一种数据结构，形象的说就是**索引是数据的目录**。

### 索引的分类

- 按「数据结构」分类：**B+tree索引、Hash索引、Full-text索引**。
- 按「物理存储」分类：**聚簇索引（主键索引）、二级索引（辅助索引）**。
- 按「字段特性」分类：**主键索引、唯一索引、普通索引、前缀索引**。
- 按「字段个数」分类：**单列索引、联合索引**。

#### 按数据结构分类

从数据结构的角度来看，MySQL 常见索引有 B+Tree 索引、HASH 索引、Full-Text 索引。

InnoDB 是在 MySQL 5.5 之后成为默认的 MySQL 存储引擎。

在创建表时，InnoDB 存储引擎会根据不同的场景选择不同的列作为索引：

- 如果有主键，默认会使用主键作为聚簇索引的索引键（key）；
- 如果没有主键，就选择第一个不包含 NULL 值的唯一列作为聚簇索引的索引键（key）；
- 在上面两个都没有的情况下，InnoDB 将自动生成一个隐式自增 id 列作为聚簇索引的索引键（key）；

其它索引都属于辅助索引（Secondary Index），也被称为二级索引或非聚簇索引。**创建的主键索引和二级索引默认使用的是 B+Tree 索引**。

- 主键索引的叶子节点存储的是具体的数据。

- 二级索引的叶子节点不会存储数据而是存储主键值，然后再通过回表的方式查询到具体数据。

#### 按物理存储分类

从物理存储的角度来看，索引分为聚簇索引（主键索引）、二级索引（辅助索引）。

- 主键索引的 B+Tree 的叶子节点存放的是实际数据，所有完整的用户记录都存放在主键索引的 B+Tree 的叶子节点里；
- 二级索引的 B+Tree 的叶子节点存放的是主键值，而不是实际数据。

注意：在二级索引时如果要查询的是主键ID就不需要再进行回表操作。

#### 按字段特性分类

从字段特性的角度来看，索引分为主键索引、唯一索引、普通索引、前缀索引。

- 主键索引
- 唯一索引

​		唯一索引建立在 UNIQUE 字段上的索引，一张表可以有多个唯一索引，索引列的值必须唯一，但是允许有空值。

```sql
CREATE TABLE table_name  (
  ....
  UNIQUE KEY(index_column_1,index_column_2,...) 
);
```



- 普通索引

  普通索引就是建立在普通字段上的索引，既不要求字段为主键，也不要求字段为 UNIQUE

  ```sql
  CREATE TABLE table_name  (
    ....
    INDEX(index_column_1,index_column_2,...) 
  );
  ```

- 前缀索引

​		前缀索引是指对字符类型字段的前几个字符建立的索引，而不是在整个字段上建立的索引，前缀索引可以建立在字段类型为 char、 varchar、binary、varbinary 的列上。

使用前缀索引的目的是为了减少索引占用的存储空间，提升查询效率。

```sql
CREATE TABLE table_name(
    column_list,
    INDEX(column_name(length))
); 
```

### 按字段个数分类

从字段个数的角度来看，索引分为单列索引、联合索引（复合索引）。

- 建立在单列上的索引称为单列索引，比如主键索引；
- 建立在多列上的索引称为联合索引；

##### 联合索引

通过将多个字段组合成一个索引，该索引就被称为联合索引。比如将商品表中的 product_no 和 name 字段组合成联合索引`(product_no, name)`，创建联合索引的方式如下：

```sql
CREATE INDEX index_product_no_name ON product(product_no, name);
```

## 什么时候需要 / 不需要创建索引？

索引最大的好处是提高查询速度，但是索引也是有缺点的，比如：

- 需要占用物理空间，数量越大，占用空间越大；
- 创建索引和维护索引要耗费时间，这种时间随着数据量的增加而增大；
- 会降低表的增删改的效率，因为每次增删改索引，B+ 树为了维护索引有序性，都需要进行动态维护。

所以，索引不是万能钥匙，它也是根据场景来使用的。

#### 什么时候适用索引？

- 字段有唯一性限制的，比如商品编码；
- 经常用于 `WHERE` 查询条件的字段，这样能够提高整个表的查询速度，如果查询条件不是一个字段，可以建立联合索引。
- 经常用于 `GROUP BY` 和 `ORDER BY` 的字段，这样在查询的时候就不需要再去做一次排序了，因为我们都已经知道了建立索引之后在 B+Tree 中的记录都是排序好的。

#### 什么时候不需要创建索引？

- `WHERE` 条件，`GROUP BY`，`ORDER BY` 里用不到的字段，索引的价值是快速定位，如果起不到定位的字段通常是不需要创建索引的，因为索引是会占用物理空间的。
- 字段中存在大量重复数据，不需要创建索引，比如性别字段，只有男女，如果数据库表中，男女的记录分布均匀，那么无论搜索哪个值都可能得到一半的数据。在这些情况下，还不如不要索引，因为 MySQL 还有一个查询优化器，查询优化器发现某个值出现在表的数据行中的百分比很高的时候，它一般会忽略索引，进行全表扫描。
- 表数据太少的时候，不需要创建索引；
- 经常更新的字段不用创建索引，比如不要对电商项目的用户余额建立索引，因为索引字段频繁修改，由于要维护 B+Tree的有序性，那么就需要频繁的重建索引，这个过程是会影响数据库性能的。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/mysql/%E7%B4%A2%E5%BC%95/%E7%B4%A2%E5%BC%95%E6%80%BB%E7%BB%93.drawio.png)



## MySQL 事务

### 事务的特性

一般来说，事务是必须满足4个条件（ACID）：：原子性（**A**tomicity，或称不可分割性）、一致性（**C**onsistency）、隔离性（**I**solation，又称独立性）、持久性（**D**urability）。

- **原子性：**一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。
- **一致性：**在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。
- **隔离性：**数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。
- **持久性：**事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

### 并行事务引发的问题

MySQL服务端允许多个客户端连接，这意味着MYSQL可能同时处理多个事务的情况，并行事务就可能引发**脏读（dirty read）、不可重复读（non-repeatable read）、幻读（phantom read）的问题**。

#### 脏读（事务未提交）

**如果一个事务「读到」了另一个「未提交事务修改过的数据」，就意味着发生了「脏读」现象。**

也就是事务A修改数据之后未提交，与此同时事务B 获取了数据，此时数据时错误的，也就是脏读。

#### 不可重复读（事务提交）

**在一个事务内多次读取同一个数据，如果出现前后两次读到的数据不一样的情况，就意味着发生了「不可重复读」现象。**

事务A在执行事务中有两次读取数据，在这两次读取之间，事务B恰好修改了此数据，并提交。

#### 幻读

**在一个事务内多次查询某个符合查询条件的「记录数量」，如果出现前后两次查询到的记录数量不一样的情况，就意味着发生了「幻读」现象。**



注：幻读和不可重复读的区别在于：不可重复读是锁定满足条件的记录，幻读是锁定满足条件及其相近的记录。

### 事务的隔离级别

- 脏读：读到其他事务未提交的数据；
- 不可重复读：前后读取的数据不一致；
- 幻读：前后读取的记录数量不一致。

三种现象的严重性：

​	脏读 > 不可重复读 > 幻读

SQL 标准提出了四种隔离级别来规避这些现象，隔离级别越高，性能效率就越低，这四个隔离级别如下：

- **读未提交（\*read uncommitted\*）**，指一个事务还没提交时，它做的变更就能被其他事务看到；
- **读已提交（\*read committed\*）**，指一个事务提交之后，它做的变更才能被其他事务看到；
- **可重复读（\*repeatable read\*）**，指一个事务执行过程中看到的数据，一直跟这个事务启动时看到的数据是一致的，**MySQL InnoDB 引擎的默认隔离级别**；
- **串行化（\*serializable\* ）**；会对记录加上读写锁，在多个事务对这条记录进行读写操作时，如果发生了读写冲突的时候，后访问的事务必须等前一个事务执行完成，才能继续执行；

隔离水平由高到低：

​	串行化 > 可重复读 > 读已提交 > 读未提交

不通隔离级别可能发生的现象：

- 读未提交：脏读 、 不可重复读、 幻读
- 读已提交：不可重复读、 幻读
- 可重复读：幻读
- 串行化：

这四种隔离级别具体是如何实现的呢？

- 对于「读未提交」隔离级别的事务来说，因为可以读到未提交事务修改的数据，所以直接读取最新的数据就好了；
- 对于「串行化」隔离级别的事务来说，通过加读写锁的方式来避免并行访问；
- 对于「读提交」和「可重复读」隔离级别的事务来说，它们是通过 **Read View \**来实现的，它们的区别在于创建 Read View 的时机不同，大家可以把 Read View 理解成一个数据快照，就像相机拍照那样，定格某一时刻的风景。\**「读提交」隔离级别是在「每个语句执行前」都会重新生成一个 Read View，而「可重复读」隔离级别是「启动事务时」生成一个 Read View，然后整个事务期间都在用这个 Read View**。

#### Read View ???

对于「读提交」和「可重复读」隔离级别的事务来说，它们是通过 Read View 来实现的，它们的区别在于创建 Read View 的时机不同：

- 「读提交」隔离级别是在每个 select 都会生成一个新的 Read View，也意味着，事务期间的多次读取同一条数据，前后两次读的数据可能会出现不一致，因为可能这期间另外一个事务修改了该记录，并提交了事务。
- 「可重复读」隔离级别是启动事务时生成一个 Read View，然后整个事务期间都在用这个 Read View，这样就保证了在事务期间读到的数据都是事务启动前的记录。

这两个隔离级别实现是通过「事务的 Read View 里的字段」和「记录中的两个隐藏列」的比对，来控制并发事务访问同一个记录时的行为，这就叫 MVCC（多版本并发控制）。



### 事务控制语句：

- BEGIN 或 START TRANSACTION 显式地开启一个事务；
- COMMIT 也可以使用 COMMIT WORK，不过二者是等价的。COMMIT 会提交事务，并使已对数据库进行的所有修改成为永久性的；
- ROLLBACK 也可以使用 ROLLBACK WORK，不过二者是等价的。回滚会结束用户的事务，并撤销正在进行的所有未提交的修改；
- SAVEPOINT identifier，SAVEPOINT 允许在事务中创建一个保存点，一个事务中可以有多个 SAVEPOINT；
- RELEASE SAVEPOINT identifier 删除一个事务的保存点，当没有指定的保存点时，执行该语句会抛出一个异常；
- ROLLBACK TO identifier 把事务回滚到标记点；
- SET TRANSACTION 用来设置事务的隔离级别。InnoDB 存储引擎提供事务的隔离级别有READ UNCOMMITTED、READ COMMITTED、REPEATABLE READ 和 SERIALIZABLE。

## MySQL 锁

### mysql锁有哪些

#### 全局锁

> 怎么用全局锁？

```sql
flush tables with read lock
```

执行这条语句之后，整个数据库就处于只读的状态，以下操作都会被阻塞：

- 对数据的增删改：insert、delete、update等
- 对表结构的更改，alter table、 drop table 等

释放锁：

```sql
unlock tables
```

另外，当连接断开，全局锁会被自动释放。

> 全局锁应用的场景？

做全库逻辑备份

> 备份数据库，怎么样不影响业务？

在数据库是“可重复读的隔离级别”的时候，是可以实现的。

备份数据库使用 mysqldump工具， 加上 -single-transaction 参数就在备份数据库之前开启事务。仅支持上边提到的 可重复读的隔离级别。

#### 表级锁

> 表级锁有哪些？怎么使用？

- 表锁；
- 元数据锁（MDL）；
- 意向锁；
- AUTO-INC锁；

##### 表锁

```sql
//表级别的共享锁， 也就是读锁
lock tables t_student read;
//表级别的独占锁，也就是写锁
lock tables t_student write;
//释放锁
unlock tables
```

<u>少用表锁，颗粒度太大，也就是上锁的数据量或者说范围太大，一般使用后边提到的行锁。</u>

##### 元数据锁（MDL）

默认mysql会隐示的使用MDL，对表操作时会加上这个锁：

- 对一张表进行CRUD操作时，加的是MDL读锁；
- 对一张表做结构改变操作的时候，加的是MDL写锁；

MDL是为了防止，对当前表CRUD操作时其他线程也对表进行变更。

<font color=red>注意：当一个长事务对表进行操作时，如果仅仅是select操作，上的是读锁，但后边等待的队列中只要有一个是需要上写锁的操作，就会导致队列中所有读锁都会一直阻塞下去，因为写锁的优先级要高于读锁</font>

##### 意向锁

- 在使用 InnoDB 引擎的表里对某些记录加上「共享锁」之前，需要先在表级别加上一个「意向共享锁」；
- 在使用 InnoDB 引擎的表里对某些纪录加上「独占锁」之前，需要先在表级别加上一个「意向独占锁」；

也就是，当执行插入、更新、删除操作，需要先对表加上「意向独占锁」，然后对该记录加独占锁。

而普通的 select 是不会加行级锁的，普通的 select 语句是利用 MVCC 实现一致性读，是无锁的。

不过，select 也是可以对记录加共享锁和独占锁的，具体方式如下:

```sql
//先在表上加上意向共享锁，然后对读取的记录加共享锁
select ... lock in share mode;

//先表上加上意向独占锁，然后对读取的记录加独占锁
select ... for update;
```

<font color=red>意向共享锁和意向独占锁是表级锁，不会和行级的共享锁和独占锁发生冲突，而且意向锁之间也不会发生冲突，只会和共享表锁（lock tables ... read）和独占表锁（lock tables ... write）发生冲突。</font>

表锁和行锁是满足读读共享、读写互斥、写写互斥的。

**意向锁的目的是为了快速判断表里是否有记录被加锁**。??

##### AUTO-INC锁

在为某个字段声明 `AUTO_INCREMENT` 属性时，之后可以在插入数据时，可以不指定该字段的值，数据库会自动给该字段赋值递增的值，这主要是通过 AUTO-INC 锁实现的。

AUTO-INC 锁是特殊的表锁机制，锁**不是再一个事务提交后才释放，而是再执行完插入语句后就会立即释放**。

但是， AUTO-INC 锁再对大量数据进行插入的时候，会影响插入性能，因为另一个事务中的插入会被阻塞。

因此， 在 MySQL 5.1.22 版本开始，InnoDB 存储引擎提供了一种**轻量级的锁**来实现自增。

一样也是在插入数据的时候，会为被 `AUTO_INCREMENT` 修饰的字段加上轻量级锁，**然后给该字段赋值一个自增的值，就把这个轻量级锁释放了，而不需要等待整个插入语句执行完后才释放锁**。

InnoDB 存储引擎提供了个 innodb_autoinc_lock_mode 的系统变量，是用来控制选择用 AUTO-INC 锁，还是轻量级的锁。

- 当 innodb_autoinc_lock_mode = 0，就采用 AUTO-INC 锁；
- 当 innodb_autoinc_lock_mode = 2，就采用轻量级锁；
- 当 innodb_autoinc_lock_mode = 1，这个是默认值，两种锁混着用，如果能够确定插入记录的数量就采用轻量级锁，不确定时就采用 AUTO-INC 锁。

不过，当 innodb_autoinc_lock_mode = 2 是性能最高的方式，但是会带来一定的问题。因为并发插入的存在，在每次插入时，自增长的值可能不是连续的，**这在有主从复制的场景中是不安全的**。

#### 行级锁

行级锁的类型主要有三类：

- Record Lock，记录锁，也就是仅仅把一条记录锁上；
- Gap Lock，间隙锁，锁定一个范围，但是不包含记录本身；
- Next-Key Lock：Record Lock + Gap Lock 的组合，锁定一个范围，并且锁定记录本身。

前面也提到，普通的 select 语句是不会对记录加锁的，如果要在查询时对记录加行锁，可以使用下面这两个方式：

```sql
//对读取的记录加共享锁
select ... lock in share mode;

//对读取的记录加独占锁
select ... for update;
```

上面这两条语句必须在一个事务中，**因为当事务提交了，锁就会被释放**，所以在使用这两条语句的时候，要加上 begin、start transaction 或者 set autocommit = 0。

### mysql是怎么加锁的？

解析上边提到的行级锁，不同mysql版本对于加锁的表现不同，一下是基于**mysql8.0.26**

对记录加锁时，**加锁的基本单位是 next-key lock**，它是由记录锁和间隙锁组合而成的，**next-key lock 是前开后闭区间，而间隙锁是前开后开区间**。

#### 唯一索引等值查询

当我们用唯一索引进行等值查询的时候，查询的记录存不存在，加锁的规则也会不同：

- **当查询的记录是存在的，在用「唯一索引进行等值查询」时，next-key lock 会退化成「记录锁」**。
- **当查询的记录是不存在的，在用「唯一索引进行等值查询」时，next-key lock 会退化成「间隙锁」**。

#### 唯一索引范围查询

- 唯一索引在满足一些条件的时候，next-key lock 退化为间隙锁和记录锁。
- 非唯一索引范围查询，next-key lock 不会退化为间隙锁和记录锁。

#### 非唯一索引等值查询

当我们用非唯一索引进行等值查询的时候，查询的记录存不存在，加锁的规则也会不同：

- **当查询的记录存在时，除了会加 next-key lock 外，还额外加间隙锁，也就是会加两把锁**。
- **当查询的记录不存在时，只会加 next-key lock，然后会退化为间隙锁，也就是只会加一把锁。**

### update没加索引会锁全表？

**在 update 语句的 where 条件没有使用索引，就会全表扫描，于是就会对所有记录加上 next-key 锁（记录锁 + 间隙锁），相当于把整个表锁住了**。

> 如何避免？

mysql中的 sql_safe_updates 设置为1时。

update语句需要满足以下条件之一：

- 使用where，并且where条件中必须有索引列；
- 使用 limit；
- 同时使用where和limit，此时where可以没有索引；

### mysql死锁，怎么办？

#### 死锁的产生

本次案例使用存储引擎 Innodb，隔离级别为可重复读（RR）。

详情 从小林coding 图解mysql中

使用了 select ... from .. for update; 为了避免不使用 独占锁的情况下出现幻读。

导致 事务A和B的insert语句都处于阻塞状态。

#### 如何避免死锁？

死锁的四个必要条件：**互斥、占有且等待、不可强占用、循环等待**。

在数据库层面，有两种策略通过「打破循环等待条件」来解除死锁状态：

- 设置事务等待锁的超时时间：innodb_lock_wait_timeout 默认50秒
- 开启主动死锁检测：主动死锁检测在发现死锁后，主动回滚死锁链条中的某一个事务，让其他事务得以继续执行。将参数 `innodb_deadlock_detect` 设置为 on，表示开启这个逻辑，默认就开启。

## MySQL日志

三种日志：

- undo log（回滚日志）：是 innodb存储引擎层生成的日志，实现事务的原子性，主要用于事务的回滚和MVCC。

- redo log （重做日志）：是innodb存储引擎层生成的日志，实现事务的持久性，主要用于掉电等故障恢复。

- binlog（归档日志）：是server层生成的日志，主要用于数据备份和主从复制。

  

具体更新一条记录 `UPDATE t_user SET name = 'xiaolin' WHERE id = 1;` 的流程如下:

1. 执行器负责具体执行，会调用存储引擎的接口，通过主键索引树搜索获取 id = 1 这一行记录：
   - 如果 id=1 这一行所在的数据页本来就在 buffer pool 中，就直接返回给执行器更新；
   - 如果记录不在 buffer pool，将数据页从磁盘读入到 buffer pool，返回记录给执行器。
2. 执行器得到聚簇索引记录后，会看一下更新前的记录和更新后的记录是否一样：
   - 如果一样的话就不进行后续更新流程；
   - 如果不一样的话就把更新前的记录和更新后的记录都当作参数传给 InnoDB 层，让 InnoDB 真正的执行更新记录的操作；
3. 开启事务， InnoDB 层更新记录前，首先要记录相应的 undo log，因为这是更新操作，需要把被更新的列的旧值记下来，也就是要生成一条 undo log，undo log 会写入 Buffer Pool 中的 Undo 页面，不过在修改该 Undo 页面前需要先记录对应的 redo log，所以**先记录修改 Undo 页面的 redo log ，然后再真正的修改 Undo 页面**。
4. InnoDB 层开始更新记录，根据 WAL 技术，**先记录修改数据页面的 redo log ，然后再真正的修改数据页面**。修改数据页面的过程是修改 Buffer Pool 中数据所在的页，然后将其页设置为脏页，为了减少磁盘I/O，不会立即将脏页写入磁盘，后续由后台线程选择一个合适的时机将脏页写入到磁盘。
5. 至此，一条记录更新完了。
6. 在一条更新语句执行完成后，然后开始记录该语句对应的 binlog，此时记录的 binlog 会被保存到 binlog cache，并没有刷新到硬盘上的 binlog 文件，在事务提交时才会统一将该事务运行过程中的所有 binlog 刷新到硬盘。
7. 事务提交（为了方便说明，这里不说组提交的过程，只说两阶段提交）：
   - **prepare 阶段**：将 redo log 对应的事务状态设置为 prepare，然后将 redo log 刷新到硬盘；
   - **commit 阶段**：将 binlog 刷新到磁盘，接着调用引擎的提交事务接口，将 redo log 状态设置为 commit（将事务设置为 commit 状态后，刷入到磁盘 redo log 文件）；
8. 至此，一条更新语句执行完成。

## MySQL 内存

Innodb 存储引擎设计了一个**缓冲池（\*Buffer Pool\*）**，来提高数据库的读写性能。

Buffer Pool 以页为单位缓冲数据，可以通过 `innodb_buffer_pool_size` 参数调整缓冲池的大小，默认是 128 M。

Innodb 通过三种链表来管理缓页：

- Free List （空闲页链表），管理空闲页；
- Flush List （脏页链表），管理脏页；
- LRU List，管理脏页+干净页，将最近且经常查询的数据缓存在其中，而不常查询的数据就淘汰出去。；

InnoDB 对 LRU 做了一些优化，我们熟悉的 LRU 算法通常是将最近查询的数据放到 LRU 链表的头部，而 InnoDB 做 2 点优化：

- 将 LRU 链表 分为**young 和 old 两个区域**，加入缓冲池的页，优先插入 old 区域；页被访问时，才进入 young 区域，目的是为了解决预读失效的问题。
- 当**「页被访问」且「 old 区域停留时间超过 `innodb_old_blocks_time` 阈值（默认为1秒）」**时，才会将页插入到 young 区域，否则还是插入到 old 区域，目的是为了解决批量数据访问，大量热数据淘汰的问题。

可以通过调整 `innodb_old_blocks_pc` 参数，设置 young 区域和 old 区域比例。

在开启了慢 SQL 监控后，如果你发现「偶尔」会出现一些用时稍长的 SQL，这可因为脏页在刷新到磁盘时导致数据库性能抖动。如果在很短的时间出现这种现象，就需要调大 Buffer Pool 空间或 redo log 日志的大小。

------

# Mysql试题

[试题-掘金](https://juejin.cn/post/6844903824935632909)

[github mysql](https://github.com/caokegege/Interview/blob/master/db/%E6%9C%80%E5%85%A8MySQL%E9%9D%A2%E8%AF%9560%E9%A2%98%E5%92%8C%E7%AD%94%E6%A1%88.md)

