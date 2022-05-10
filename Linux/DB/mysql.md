# 安装

## 下载安装包

官网下载rpm包

```
mysql-community-common-5.7.38-1.el7.x86_64.rpm 
mysql-community-libs-5.7.38-1.el7.x86_64.rpm 
mysql-community-client-5.7.38-1.el7.x86_64.rpm
mysql-community-server-5.7.38-1.el7.x86_64.rpm
yum install mysql-community-common-5.7.38-1.el7.x86_64.rpm mysql-community-libs-5.7.38-1.el7.x86_64.rpm mysql-community-client-5.7.38-1.el7.x86_64.rpm mysql-community-server-5.7.38-1.el7.x86_64.rpm -y
```

## 初始化

注意：mysql 5.7初始使用了随机密码，安装后查看/var/log/musqld.log，搜索password查看初始密码

```
# 登录数据库后，需修改root密码，才能对数据库进行操作
mysql> alter user root@localhost identified by 'new_passwd'

# 创建并授权用户
# grant 权限 on database.table to 'username'@'host' identified by 'passwd';
mysql> grant all on *.* to 'dev'@'%' identified by 'passwd';
mysql> flush privileges;

# 移除权限
mysql> revoke all on *.* from username;
```

# SQL语句

## DML——CRUD 增删改查

### insert

```sql
-- 向表中插入一行数据，自增字段、缺省值字段、可为空字段可以不写
insert into table_name (col_name, ...) values (value1, ...);

-- 将select查询的结果插入到表中，查询字段和插入字段相同
insert into table_name select ...;

-- 如果主键或唯一键冲突就执行update后的设置 
insert into table_name (col_name, ...) values (value1, ....) on DUPLICATE key update col_name1=value1, ...;

-- 如果主键、唯一键冲突就忽略错误，返回一个警告
insert ignore into table_name (col_name, ...) values (value1, ...);
```

### update

```sql
-- 更新语句一定要加条件，否则会更细所有数据
update table_name set col_name1 = 'value1' where col_name = 'value';
```

### delete

```sql
-- 删除符合条件的记录，切记加条件
delete from table_name where col_name = 'value';
```

### select

#### limit子句

```sql
-- limit 2, 5等于limit 5 offset 2
select emp_no as id, concat(first_name, ' ', last_name) as name from employees as emp where emp.emp_no > 10002 limit 2, 5;
select emp_no as id, concat(first_name, ' ', last_name) as name from employees as emp where emp.emp_no > 10002 limit 5 offset 2;
```

#### where子句

| 运算符       | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| =            | 等于                                                         |
| <>           | 不等于                                                       |
| >、<、>=、<= | 大于、小于、大于等于、小于等于                               |
| BETWEEN      | 在某个范围之内，between a and b等价于[a, b]                  |
| LIKE         | 字符串模式匹配，%表示任意多个字符， _表示一个字符<br />LIKE忽略大小写；LIKE BINARY区分大小写 |
| IN           | 指定针对某个列的多个可能值                                   |
| AND          | 与                                                           |
| OR           | 或                                                           |

注意：如果很多表达式需要使用AND、OR计算逻辑表达式的值的时候，由于有结合律的问题，建议使用小括号来避免产生错误

```sql
-- 条件查询
select * from tbl_name where id in (1001, 1002, 1003);
```

#### order by子句

对查询结果进行排序，升序ASC，降序DESC

先以第一个字段排序，第一个字段相同再以第二个字段排序

```sql
-- 降序
select * form tbl_name order by emp_no, dept_no desc;
```

#### distinct

去除重复记录

```sql
select distinct dept_no from tbl_name;
```

#### 聚合函数

| 函数                            | 描述                                                   |
| ------------------------------- | ------------------------------------------------------ |
| count(expr)                     | 返回记录中记录的数目，如果指定列，则返回非NULL值的行数 |
| count(distinct expr, [expr...]) | 返回不重复的非NULL值的函数                             |
| avg([distinct] expr)            | 返回平均值，返回不同值的平均值                         |
| min(expr), max(expr)            | 最小值， 最大值                                        |
| sum([distinct] expr)            | 求和，Distinct返回不同值求和                           |

```sql
-- 聚合函数
select count(*), avg(emp_no), sum(emp_no), min(emp_no), max(emp_no) from tbl_name;
```

#### 分组查询

使用group by子句，如果有条件，使用having子句过滤分组、聚合过的结果

```sql
-- 使用别名， having子句对分组结果过滤
select emp_no, avg(salary) as sal_avg from tbl_name group by emp_no having sal_avg > 4500;
```

#### 子查询

查询语句可以嵌套，内部查询就是子查询

子查询必须在一组小括号中

子查询中不能使用order by

```sql
select * from tbl_name where emp_no in (select emp_no from tbl_name where emp_no > 1005) order by emp_no;
```

#### 连接join

交叉连接cross join，笛卡尔乘积，全部交叉

在MySQL中，cross join从语法上说与inner join等同

```sql
-- 交叉连接
select * from employees cross join salaries;
-- 隐式连接
select * from employees, salaries;
```

**内连接**

inner join

等值连接，只选某些field相等的元组(行)，使用on限定关联的结果

自然连接，特殊的等值连接，会去掉重复的列

```sql
-- on等值连接
select * from employees join salaries on employees.emp_no = salaries.emp_no;

-- 自然连接，去掉重复列。且自行使用employees.emp_no = salaries.emp_no的条件
select * from employees natural join salaries;
```

**外连接**

分为左连接；右连接；全外连接

```sql
-- 左连接
select * from employees left join salaries on employees.emp_no = salaries.emp_no;

-- 右连接
select * from employees right join salaries on employees.emp_no = salaries.emp_no;
```

**自连接**

假设有表manager，字段和记录如下

| emp_no | name  | manager_no |
| ------ | ----- | ---------- |
| 1      | tom   |            |
| 2      | jerry | 1          |
| 3      | ben   | 2          |

```sql
-- 有领导的员工
select * from manager where manager_no is not null;

-- 所有有领导的员工及其领导的名字
select worker.emp_no, worker.name, mgr.name as leaderName from manager as worker join manager as mgr on worker.manager_no = mgr.emp_no
```
