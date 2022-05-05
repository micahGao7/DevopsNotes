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

