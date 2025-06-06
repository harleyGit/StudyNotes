> <h1 id=""></h1>
- [**约束**](#约束)
	- [约束分类](#约束分类) 
	- [约束作用范围](#约束作用范围) 
	- [约束的作用](#约束的作用)
		- [not null非空约束](#notnull非空约束) 
			- [建表后添加约束](#建表后添加约束) 
				- [添加非空约束](#添加非空约束) 
				- [删除非空约束](#删除非空约束)
		- [唯一性约束unique](#唯一性约束unique)
			- [添加唯一约束](#添加唯一约束) 
			- [列级、表级约束](#列级、表级约束) 
			- [建表后添加唯一键约束](#建表后添加唯一键约束) 
			- [关于复合唯一约束](#关于复合唯一约束) 
			- [删除唯一约束](#删除唯一约束) 
		- [主键约束primary key](#主键约束primarykey) 
			- [添加主键约束](#添加主键约束)
			- [主键-列级约束](#主键-列级约束) 
			- [主键-表级约束](#主键-表级约束) 
			- [建表后增加主键约束](#建表后增加主键约束) 
			- [主键-复合约束](#主键-复合约束) 
			- [删除主键约束](#删除主键约束)
		- [自增列：AUTO_INCREMENT](#自增列：AUTO_INCREMENT) 
			- [如何指定自增约束](#如何指定自增约束) 
			- [建表后添加auto_increment](#建表后添加auto_increment) 
			- [如何删除自增约束](#如何删除自增约束) 
			- [MySQL8.0新特性—自增变量的持久化](#MySQL8.0新特性—自增变量的持久化)
		- [外键约束foreign key](#外键约束foreignkey) 
			- [添加外键约束](#添加外键约束) 
			- [建表后添加外键约束](#建表后添加外键约束)
			- [外键-演示问题](#外键-演示问题)	
				- [失败：不是键列](#失败：不是键列 ) 
					- [demo-主表没有添加主键约束](#demo-主表没有添加主键约束)
				- [添加、删除、修改问题 ](#添加、删除、修改问题)
					- [demo-插入失败](#demo-插入失败)
					- [demo-删除失败](#demo-删除失败) 
					- [demo-修改主表数据失败](#demo-修改主表数据失败)
			- [约束等级](#约束等级)
			- [删除外键约束](#删除外键约束)
			- [外键-开发场景](#外键-开发场景)
		- [检查约束check](#检查约束check) 
		- [默认值约束](#默认值约束)
		- [添加/删除约束](#添加/删除约束)
		- [查看表中所有约束](#查看表中所有约束)
	- [**约束-思考题**](#约束-思考题)
- [**视图**](#视图)
	- [创建视图](#创建视图) 
	- [创建单表视图](#创建单表视图) 
	- [查看视图](#查看视图) 
	- [更新视图的数据](#更新视图的数据) 
	- [不可更新的视图](#不可更新的视图)



<br/><br/><br/>

***
<br/>

> <h1 id="约束">约束</h1>

**为什么需要约束**

数据完整性（Data Integrity）是指数据的精确性（Accuracy）和可靠性（Reliability）。它是防止数据库中存在不符合语义规定的数据和防止因错误信息的输入输出造成无效操作或错误信息而提出的。

- 为了保证数据的完整性，SQL规范以约束的方式对**表数据进行额外的条件限制**。从以下四个方面考虑：

	- `实体完整性（Entity Integrity）`：例如，同一个表中，不能存在两条完全相同无法区分的记录
	- `域完整性（Domain Integrity）`：例如：年龄范围0-120，性别范围“男/女”
	- `引用完整性（Referential Integrity）`：例如：员工所在部门，在部门表中要能找到这个部门
	- `用户自定义完整性（User-defined Integrity）`：例如：用户名唯一、密码不能为空等，本部门经理的工资不得高于本部门职工的平均工资的5倍。


***
<br/><br/><br/>

> <h2 id="约束分类">约束分类</h2>
<br/>

> <h2 id="notnull非空约束">not null非空约束</h3>

**`关键字:NOT NULL`**

<br/>

- **特点**
	- 默认，所有的类型的值都可以是NULL，包括INT、FLOAT等数据类型
	- 非空约束只能出现在表对象的列上，只能某个列单独限定非空，不能组合非空
	- 一个表可以有很多列都分别限定了非空
	- 空字符串''不等于NULL，0也不等于NULL

<br/>

 **添加非空约束**

**（1）建表时**

```mysql
CREATE TABLE 表名称(
	字段名  数据类型,
    字段名  数据类型 NOT NULL,  
    字段名  数据类型 NOT NULL
);
```

<br/>

**举例1️⃣：**

```mysql
CREATE TABLE IF NOT EXISTS test1(
id INT NOT NULL,
last_name VARCHAR(20) NOT NULL,
email VARCHAR(25),
salary DECIMAL(10, 2)
);


desc test1;
+-----------+---------------+------+-----+---------+-------+
| Field     | Type          | Null | Key | Default | Extra |
+-----------+---------------+------+-----+---------+-------+
| id        | int           | NO   |     | NULL    |       |
| last_name | varchar(20)   | NO   |     | NULL    |       |
| email     | varchar(25)   | YES  |     | NULL    |       |
| salary    | decimal(10,2) | YES  |     | NULL    |       |
+-----------+---------------+------+-----+---------+-------+
4 rows in set (0.00 sec)
```

<br/>

**插入数据:**

```sql
INSERT INTO test1(id, last_name, email, salary)
VALUES(1, 'Tom', 'tom@126.com', 3400)


select * from test1;
+----+-----------+-------------+---------+
| id | last_name | email       | salary  |
+----+-----------+-------------+---------+
|  1 | Tom       | tom@126.com | 3400.00 |
+----+-----------+-------------+---------+
1 row in set (0.01 sec)
```

<br/>

**插入空值报错:**

```sql
INSERT INTO test1(id, last_name, email, salary)
VALUES(2, NULL, 'tom@333.com', 3400)

ERROR 1048 (23000): Column 'last_name' cannot be null
```


<br/><br/>

**举例2️⃣：**

```mysql
CREATE TABLE student(
	sid int,
    sname varchar(20) not null,
    tel char(11) ,
    cardid char(18) not null
);
```

```mysql
insert into student values(1,'张三','13710011002','110222198912032545'); #成功

insert into student values(2,'李四','13710011002',null);#身份证号为空
ERROR 1048 (23000): Column 'cardid' cannot be null

insert into student values(2,'李四',null,'110222198912032546');#成功，tel允许为空

insert into student values(3,null,null,'110222198912032547');#失败
ERROR 1048 (23000): Column 'sname' cannot be null
```

<br/><br/>
> <h3 id="建表后添加约束">建表后添加约束</h3>

```mysql
alter table 表名称 modify 字段名 数据类型 not null;
```

<br/><br/>
> <h3 id="添加非空约束">添加非空约束</h3>

```mysql
ALTER TABLE emp
MODIFY sex VARCHAR(30) NOT NULL;


alter table student modify sname varchar(20) not null;
```

<br/><br/>
> <h3 id="删除非空约束">删除非空约束</h3>

```mysql
alter table 表名称 modify 字段名 数据类型 NULL;#去掉not null，相当于修改某个非注解字段，该字段允许为空

或 

alter table 表名称 modify 字段名 数据类型;#去掉not null，相当于修改某个非注解字段，该字段允许为空
```

```mysql
ALTER TABLE emp
MODIFY sex VARCHAR(30) NULL;


ALTER TABLE emp
MODIF
```


***
<br/><br/><br/>

> <h2 id="唯一性约束unique">唯一性约束unique</h2>
用来限制某个字段/某列的值不能重复。

**关键字: UNIQUE**

<br/>

- **特点**
	- 同一个表可以有多个唯一约束。
	- 唯一约束可以是某一个列的值唯一，也可以多个列组合的值唯一。
	- 唯一性约束允许列值为空(空值可以不唯一,可以多添加几个)。
	- 在创建唯一约束的时候，如果不给唯一约束命名，就默认和列名相同。
	- **MySQL会给唯一约束的列上默认创建一个唯一索引。**

<br/><br/>
> <h3 id="添加唯一约束">添加唯一约束</h3>
（1）建表时

```mysql
create table 表名称(
	字段名  数据类型,
    字段名  数据类型  unique,  
    字段名  数据类型  unique key,
    字段名  数据类型
);
create table 表名称(
	字段名  数据类型,
    字段名  数据类型,  
    字段名  数据类型,
    [constraint 约束名] unique key(字段名)
);
```

<br/><br/>
> <h3 id="列级、表级约束">列级、表级约束</h3>


```mysql
create table student(
	sid int,
    sname varchar(20),
    tel char(11) unique,# 列级约束
    cardid char(18) unique key
);
```

<br/>

```mysql
CREATE TABLE t_course(
	cid INT UNIQUE,
	cname VARCHAR(100) UNIQUE,# 列级约束 
	description VARCHAR(200)
);
```

<br/>

```mysql
CREATE TABLE USER(
 id INT NOT NULL,
 NAME VARCHAR(25),
 PASSWORD VARCHAR(16),
 
 -- 使用表级约束语法
 CONSTRAINT uk_name_pwd UNIQUE(NAME,PASSWORD)
);
```

**uk_name_pwd:** 表级约束的别名

<br/>

> 表示用户名和密码组合不能重复

```mysql
insert into student values(1,'张三','13710011002','101223199012015623');
insert into student values(2,'李四','13710011003','101223199012015624');
```

```mysql
mysql> select * from student;
+-----+-------+-------------+--------------------+
| sid | sname | tel         | cardid             |
+-----+-------+-------------+--------------------+
|   1 | 张三  | 13710011002 | 101223199012015623 |
|   2 | 李四  | 13710011003 | 101223199012015624 |
+-----+-------+-------------+--------------------+
2 rows in set (0.00 sec)
```

```mysql
insert into student values(3,'王五','13710011004','101223199012015624'); #身份证号重复
ERROR 1062 (23000): Duplicate entry '101223199012015624' for key 'cardid'

insert into student values(3,'王五','13710011003','101223199012015625'); 
ERROR 1062 (23000): Duplicate entry '13710011003' for key 'tel'
```

<br/><br/>
> <h3 id="建表后添加唯一键约束">建表后添加唯一键约束</h3>

```mysql
#字段列表中如果是一个字段，表示该列的值唯一。如果是两个或更多个字段，那么复合唯一，即多个字段的组合是唯一的
#方式1：
alter table 表名称 add unique key(字段列表); 


#方式2：
alter table 表名称 modify 字段名 字段类型 unique;
```

<br/>

**举例：**

```mysql
-- 约束名为默认字段名NAME、PASSWORD
ALTER TABLE USER 
ADD UNIQUE(NAME,PASSWORD);
```

```mysql
-- 使用关键字CONSTRAINT,指定约束名uk_name_pwd
ALTER TABLE USER 
ADD CONSTRAINT uk_name_pwd UNIQUE(NAME,PASSWORD);
-- NAME、PASSWORD中只要有一个不同即可,若是都相同就不行
```

```mysql
ALTER TABLE USER 
MODIFY NAME VARCHAR(20) UNIQUE;
```

<br/>

**举例：**

```mysql
create table student(
	sid int primary key,
	sname varchar(20),
	tel char(11) ,
	cardid char(18) 
);



alter table student add unique key(tel);

alter table student add unique key(cardid);
```

<br/><br/>
> <h3 id="关于复合唯一约束">关于复合唯一约束</h3>

```mysql
create table 表名称(
	字段名  数据类型,
	字段名  数据类型,  
	字段名  数据类型,
	unique key(字段列表) #字段列表中写的是多个字段名，多个字段名用逗号分隔，表示那么是复合唯一，即多个字段的组合是唯一的
);
```

<br/>

```mysql
#学生表
create table student(
	sid int,	#学号
	sname varchar(20),			#姓名
	tel char(11) unique key,  #电话
	cardid char(18) unique key #身份证号
);

#课程表
create table course(
	cid int,  #课程编号
	cname varchar(20)     #课程名称
);

#选课表
create table student_course(
	id int,
	sid int,
	cid int,
	score int,
	unique key(sid,cid)  #复合唯一
);
```

```mysql
insert into student values(1,'张三','13710011002','101223199012015623');#成功
insert into student values(2,'李四','13710011003','101223199012015624');#成功
insert into course values(1001,'Java'),(1002,'MySQL');#成功
```

```mysql
mysql> select * from student;
+-----+-------+-------------+--------------------+
| sid | sname | tel         | cardid             |
+-----+-------+-------------+--------------------+
|   1 | 张三  | 13710011002 | 101223199012015623 |
|   2 | 李四  | 13710011003 | 101223199012015624 |
+-----+-------+-------------+--------------------+
2 rows in set (0.00 sec)

mysql> select * from course;
+------+-------+
| cid  | cname |
+------+-------+
| 1001 | Java  |
| 1002 | MySQL |
+------+-------+
2 rows in set (0.00 sec)
```

```mysql
insert into student_course values
(1, 1, 1001, 89),
(2, 1, 1002, 90),
(3, 2, 1001, 88),
(4, 2, 1002, 56);#成功
```

```mysql
mysql> select * from student_course;
+----+------+------+-------+
| id | sid  | cid  | score |
+----+------+------+-------+
|  1 |    1 | 1001 |    89 |
|  2 |    1 | 1002 |    90 |
|  3 |    2 | 1001 |    88 |
|  4 |    2 | 1002 |    56 |
+----+------+------+-------+
4 rows in set (0.00 sec)
```

```mysql
insert into student_course values (5, 1, 1001, 88);#失败

#ERROR 1062 (23000): Duplicate entry '1-1001' for key 'sid'   违反sid-cid的复合唯一
```

<br/><br/>
> <h3 id="删除唯一约束">删除唯一约束</h3>

- 添加唯一性约束的列上也会自动创建唯一索引。
- 删除唯一约束只能通过删除唯一索引的方式删除。
- 删除时需要指定唯一索引名，唯一索引名就和唯一约束名一样。
- 如果创建唯一约束时未指定名称，如果是单列，就默认和列名相同；如果是组合列，那么默认和()中排在第一个的列名相同。也可以自定义唯一性约束名。

```mysql
SELECT * FROM information_schema.table_constraints WHERE table_name = '表名'; #查看都有哪些约束
```

```mysql
ALTER TABLE USER 
DROP INDEX uk_name_pwd;
```

> 注意：可以通过 `show index from 表名称; `查看表的索引



***
<br/><br/><br/>

> <h2 id="主键约束primarykey">主键约束primary key</h2>
**作用:** 用来唯一标识表中的一行记录。

**关键字:`primary key`**

- **特点**
	- 主键约束相当于**唯一约束+非空约束的组合**，主键约束列不允许重复，也不允许出现空值。

<br/>

- 一个表最多只能有一个主键约束，建立主键约束可以在列级别创建，也可以在表级别上创建。
- 主键约束对应着表中的一列或者多列（复合主键）
- 如果是多列组合的复合主键约束，那么这些列都不允许为空值，并且组合的值不允许重复。
- **MySQL的主键名总是PRIMARY**，就算自己命名了主键约束名也没用。
- 当创建主键约束时，系统默认会在所在的列或列组合上建立对应的**主键索引**（能够根据主键查询的，就根据主键查询，效率更高）。如果删除主键约束了，主键约束对应的索引就自动删除了。


- 需要注意的一点是，不要修改主键字段的值。因为主键是数据记录的唯一标识，如果修改了主键的值，就有可能会破坏数据的完整性。

<br/><br/>
> <h3 id="添加主键约束">添加主键约束</h3>
（1）建表时指定主键约束

```mysql
create table 表名称(
	字段名  数据类型  primary key, #列级模式
	字段名  数据类型,  
	字段名  数据类型  
);

create table 表名称(
	字段名  数据类型,
	字段名  数据类型,  
	字段名  数据类型,
	[constraint 约束名] primary key(字段名) #表级模式
);

#比如: CONSTRAINT pk_test5_id PRIMARY KEY(id)
-- pk_test5_id 主键约束别名(注意:即使你给它起了别名,它的名字还是id,不是pk_test5_id,所以没有必要起别名)
-- id: 字段名
```

<br/>

**举例：**

```mysql
create table temp(
	id int primary key,
	name varchar(20)
);
```

```mysql
mysql> desc temp;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(11)     | NO   | PRI | NULL    |       |
| name  | varchar(20) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
2 rows in set (0.00 sec)
```

```mysql
insert into temp values(1,'张三');#成功
insert into temp values(2,'李四');#成功
```

```mysql
mysql> select * from temp;
+----+------+
| id | name |
+----+------+
|  1 | 张三 |
|  2 | 李四 |
+----+------+
2 rows in set (0.00 sec)
```

```mysql
insert into temp values(1,'张三');#失败
ERROR 1062 (23000): Duplicate（重复） entry（键入，输入） '1' for key 'PRIMARY'


insert into temp values(1,'王五');#失败
ERROR 1062 (23000): Duplicate entry '1' for key 'PRIMARY'

insert into temp values(3,'张三');#成功
```

```mysql
mysql> select * from temp;
+----+------+
| id | name |
+----+------+
|  1 | 张三 |
|  2 | 李四 |
|  3 | 张三 |
+----+------+
3 rows in set (0.00 sec)
```

```mysql
insert into temp values(4,null);#成功


insert into temp values(null,'李琦');#失败
ERROR 1048 (23000): Column 'id' cannot be null
```

```mysql
mysql> select * from temp;
+----+------+
| id | name |
+----+------+
|  1 | 张三 |
|  2 | 李四 |
|  3 | 张三 |
|  4 | NULL |
+----+------+
4 rows in set (0.00 sec)
```

```mysql
#演示一个表建立两个主键约束
create table temp(
	id int primary key,
	name varchar(20) primary key
);
ERROR 1068 (42000): Multiple（多重的） primary key defined（定义）
```

<br/><br/>
> <h3 id="主键-列级约束">主键-列级约束</h3>

```mysql
CREATE TABLE emp4(
	id INT PRIMARY KEY AUTO_INCREMENT ,
	NAME VARCHAR(20)
);
```

<br/><br/>
> <h3 id="主键-表级约束">主键-表级约束</h3>

```mysql
CREATE TABLE emp5(
	id INT NOT NULL AUTO_INCREMENT,
	NAME VARCHAR(20),
	pwd VARCHAR(15),
	CONSTRAINT emp5_id_pk PRIMARY KEY(id)
);
```

<br/><br/>
> <h3 id="建表后增加主键约束">建表后增加主键约束</h3>


```mysql
ALTER TABLE 表名称 ADD PRIMARY KEY(字段列表); #字段列表可以是一个字段，也可以是多个字段，如果是多个字段的话，是复合主键
```

```mysql
ALTER TABLE student ADD PRIMARY KEY (sid);
```

```mysql
ALTER TABLE emp5 ADD PRIMARY KEY(NAME,pwd);
```

<br/><br/>
> <h3 id="主键-复合约束">主键-复合约束</h3>

```mysql
create table 表名称(
	字段名  数据类型,
	字段名  数据类型,  
	字段名  数据类型,
	primary key(字段名1,字段名2)  #表示字段1和字段2的组合是唯一的，也可以有更多个字段
);
```

```mysql
#学生表
create table student(
	sid int primary key,  #学号
	sname varchar(20)     #学生姓名
);

#课程表
create table course(
	cid int primary key,  #课程编号
	cname varchar(20)     #课程名称
);

#选课表
create table student_course(
	sid int,
	cid int,
	score int,
	primary key(sid,cid)  #复合主键
);
```

```mysql
insert into student values(1,'张三'),(2,'李四');

insert into course values(1001,'Java'),(1002,'MySQL');
```

```mysql
mysql> select * from student;
+-----+-------+
| sid | sname |
+-----+-------+
|   1 | 张三  |
|   2 | 李四  |
+-----+-------+
2 rows in set (0.00 sec)

mysql> select * from course;
+------+-------+
| cid  | cname |
+------+-------+
| 1001 | Java  |
| 1002 | MySQL |
+------+-------+
2 rows in set (0.00 sec)
```

```mysql
insert into student_course values(1, 1001, 89),(1,1002,90),(2,1001,88),(2,1002,56);
```

```mysql
mysql> select * from student_course;
+-----+------+-------+
| sid | cid  | score |
+-----+------+-------+
|   1 | 1001 |    89 |
|   1 | 1002 |    90 |
|   2 | 1001 |    88 |
|   2 | 1002 |    56 |
+-----+------+-------+
4 rows in set (0.00 sec)
```

```mysql
insert into student_course values(1, 1001, 100);
ERROR 1062 (23000): Duplicate entry '1-1001' for key 'PRIMARY'
```

```mysql
mysql> desc student_course;
+-------+---------+------+-----+---------+-------+
| Field | Type    | Null | Key | Default | Extra |
+-------+---------+------+-----+---------+-------+
| sid   | int(11) | NO   | PRI | NULL    |       |
| cid   | int(11) | NO   | PRI | NULL    |       |
| score | int(11) | YES  |     | NULL    |       |
+-------+---------+------+-----+---------+-------+
3 rows in set (0.00 sec)
```

- 再举例

```mysql
CREATE TABLE emp6(
id INT NOT NULL,
NAME VARCHAR(20),
pwd VARCHAR(15),
CONSTRAINT emp7_pk PRIMARY KEY(NAME,pwd)
);
```

<br/><br/>
> <h3 id="删除主键约束">删除主键约束(在开发中一般不会做)</h3>


```mysql
alter table 表名称 drop primary key;
```

<br/>

举例：

```mysql
ALTER TABLE student DROP PRIMARY KEY;
```

```mysql
ALTER TABLE emp5 DROP PRIMARY KEY;
```

> 说明：删除主键约束，不需要指定主键名，因为一个表只有一个主键，删除主键约束后，非空还存在。


***
<br/><br/><br/>

> <h2 id="自增列：AUTO_INCREMENT">自增列：AUTO_INCREMENT</h2>
**作用:某个字段的值自增**

**关键字:`auto_increment`**

<br/>

- **特点和要求**
	- （1）一个表最多只能有一个自增长列
	- （2）当需要产生唯一标识符或顺序值时，可设置自增长
	- （3）自增长列约束的列必须是键列（主键列，唯一键列）
	- （4）自增约束的列的数据类型必须是**整数类型**
	- （5）如果自增列指定了 0 和 null，会在当前最大值的基础上自增；如果自增列手动指定了具体值，直接赋值为具体值。

<br/>

**错误演示：**

```mysql
create table employee(
	eid int auto_increment,
	ename varchar(20)
);

# ERROR 1075 (42000): Incorrect table definition; there can be only one auto column and it must be defined as a key 

--报错原因是: eid不是一个主键约束;  
--修改为: 	eid int PRIMARY KEY auto_increment,
```

```mysql
create table employee(
	eid int primary key,
	ename varchar(20) unique key auto_increment
);

# ERROR 1063 (42000): Incorrect column specifier for column 'ename'  因为ename不是整数类型,ename是字符串的无法自己增加.
```

<br/><br/>
> <h3 id="如何指定自增约束">如何指定自增约束</h3>

**（1）建表时**

```mysql
create table 表名称(
	字段名  数据类型  primary key auto_increment,
	字段名  数据类型  unique key not null,  
	字段名  数据类型  unique key,
	字段名  数据类型  not null default 默认值, 
);


create table 表名称(
	字段名  数据类型 default 默认值 ,
	字段名  数据类型 unique key auto_increment,  
	字段名  数据类型 not null default 默认值,,
	primary key(字段名)
);
```

```mysql
create table employee(
	eid int primary key auto_increment,
	ename varchar(20)
);
```

```mysql
mysql> desc employee;
+-------+-------------+------+-----+---------+----------------+
| Field | Type        | Null | Key | Default | Extra          |
+-------+-------------+------+-----+---------+----------------+
| eid   | int(11)     | NO   | PRI | NULL    | auto_increment |
| ename | varchar(20) | YES  |     | NULL    |                |
+-------+-------------+------+-----+---------+----------------+
2 rows in set (0.00 sec)
```

<br/><br/>
> <h3 id="建表后添加auto_increment">建表后添加auto_increment</h3>


```mysql
alter table 表名称 modify 字段名 数据类型 auto_increment;
```

例如：

```mysql
create table employee(
	eid int primary key ,
	ename varchar(20)
);
```

```mysql
alter table employee modify eid int auto_increment;
```

```mysql
mysql> desc employee;
+-------+-------------+------+-----+---------+----------------+
| Field | Type        | Null | Key | Default | Extra          |
+-------+-------------+------+-----+---------+----------------+
| eid   | int(11)     | NO   | PRI | NULL    | auto_increment |
| ename | varchar(20) | YES  |     | NULL    |                |
+-------+-------------+------+-----+---------+----------------+
2 rows in set (0.00 sec)
```

<br/><br/>
> <h3 id="如何删除自增约束">如何删除自增约束</h3>


```mysql
#alter table 表名称 modify 字段名 数据类型 auto_increment;#给这个字段增加自增约束

alter table 表名称 modify 字段名 数据类型; #去掉auto_increment相当于删除
```

```mysql
alter table employee modify eid int;
```

```mysql
mysql> desc employee;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| eid   | int(11)     | NO   | PRI | NULL    |       |
| ename | varchar(20) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
2 rows in set (0.00 sec)
```

<br/><br/>
> <h3 id="MySQL8.0新特性—自增变量的持久化">MySQL8.0新特性—自增变量的持久化</h3>
在MySQL 8.0之前，自增主键AUTO_INCREMENT的值如果大于max(primary key)+1，在MySQL重启后，会重置AUTO_INCREMENT=max(primary key)+1，这种现象在某些情况下会导致业务主键冲突或者其他难以发现的问题。

下面通过案例来对比不同的版本中自增变量是否持久化。
在MySQL 5.7版本中，测试步骤如下：

创建的数据表中包含自增主键的id字段，语句如下：

```mysql
CREATE TABLE test1(
id INT PRIMARY KEY AUTO_INCREMENT
);
```

插入4个空值，执行如下：

```mysql
-- 当我们向主键(含AUTO_INCREMENT)的字段上添加0或者null时,实际上会自动的往上添加指定的序列值

INSERT INTO test1
VALUES(0),(0),(0),(0);
```

查询数据表test1中的数据，结果如下：

```mysql
mysql> SELECT * FROM test1;
+----+
| id |
+----+
|  1 |
|  2 |
|  3 |
|  4 |
+----+
4 rows in set (0.00 sec)
```

删除id为4的记录，语句如下：

```mysql
DELETE FROM test1 WHERE id = 4;
```

再次插入一个空值，语句如下：

```mysql
INSERT INTO test1 VALUES(0);
```

查询此时数据表test1中的数据，结果如下：

```mysql
mysql> SELECT * FROM test1;
+----+
| id |
+----+
|  1 |
|  2 |
|  3 |
|  5 |
+----+
4 rows in set (0.00 sec)
```

从结果可以看出，虽然删除了id为4的记录，但是再次插入空值时，并没有重用被删除的4，而是分配了5。
删除id为5的记录，结果如下：

```mysql
DELETE FROM test1 where id=5;
```

**重启数据库**，重新插入一个空值。

```mysql
INSERT INTO test1 values(0);
```

再次查询数据表test1中的数据，结果如下：

```mysql
mysql> SELECT * FROM test1;
+----+
| id |
+----+
|  1 |
|  2 |
|  3 |
|  4 |
+----+
4 rows in set (0.00 sec)
```

从结果可以看出，新插入的0值分配的是4，按照重启前的操作逻辑，此处应该分配6。出现上述结果的主要原因是自增主键没有持久化。

在MySQL 5.7系统中，对于自增主键的分配规则，是由InnoDB数据字典内部一个`计数器`来决定的，而该计数器只在`内存中维护`，并不会持久化到磁盘中。当数据库重启时，该计数器会被初始化。

在MySQL 8.0版本中，上述测试步骤最后一步的结果如下：

```mysql
mysql> SELECT * FROM test1;
+----+
| id |
+----+
|  1 |
|  2 |
|  3 |
|  6 |
+----+
4 rows in set (0.00 sec)
```

从结果可以看出，自增变量已经持久化了。

MySQL 8.0将自增主键的计数器持久化到`重做日志`中。每次计数器发生改变，都会将其写入重做日志中。如果数据库重启，InnoDB会根据重做日志中的信息来初始化计数器的内存值。



***
<br/><br/><br/>

> <h2 id="外键约束foreignkey">外键约束foreign key</h2>

**作用:限定某个表的某个字段的引用完整性。**

比如：员工表的员工所在部门的选择，必须在部门表能找到对应的部分。

![go.0.0.128.png](./../Pictures/go.0.0.128.png)

<br/>

**关键字:`FOREIGN KEY`**

- **主表和从表/父表和子表**
	- 主表（父表）：被引用的表，被参考的表
	- 从表（子表）：引用别人的表，参考别人的表

<br/>

- 例如：员工表的员工所在部门这个字段的值要参考部门表：部门表是主表，员工表是从表。

- 例如：学生表、课程表、选课表：选课表的学生和课程要分别参考学生表和课程表，学生表和课程表是主表，选课表是从表。

<br/>

**特点**

- （1）从表的外键列(也就是关联到的主表的那个字段列)，必须引用/参考主表的主键或唯一约束的列(也就是那个列必须是主键约束或者唯一约束)

**`为什么？因为被依赖/被参考的值必须是唯一的`**

- （2）在创建外键约束时，如果不给外键约束命名，**默认名不是列名，而是自动产生一个外键名**（例如 student_ibfk_1;），也可以指定外键约束名。

- （3）创建(CREATE)表时就指定外键约束的话，**先创建主表，再创建从表**

- （4）删表时，先删从表（或先删除外键约束），再删除主表

- （5）当主表的记录被从表参照时，主表的记录将不允许删除，如果要删除数据，需要先删除从表中依赖该记录的数据，然后才可以删除主表的数据

- （6）在“从表”中指定外键约束，**并且一个表可以建立多个外键约束**

- （7）从表的外键列与主表被参照的**列名字可以不相同，但是数据类型必须一样，逻辑意义一致**。如果类型不一样，创建子表时，就会出现错误`“ERROR 1005 (HY000): Can't create table'database.tablename'(errno: 150)”。`

**例如：都是表示部门编号，都是int类型。**

- （8）**当创建外键约束时，系统默认会在所在的列上建立对应的普通索引**。但是索引名是外键的约束名。（根据外键查询效率很高）

- （9）删除外键约束后，必须`手动`删除对应的索引

<br/><br/>
> <h3 id="添加外键约束">添加外键约束</h3>
（1）建表时

```mysql
create table 主表名称(
	字段1  数据类型  primary key,
	字段2  数据类型
);

create table 从表名称(
	字段1  数据类型  primary key,
	字段2  数据类型,
	[CONSTRAINT <外键约束名称>] FOREIGN KEY（从表的某个字段) references 主表名(被参考字段)
);
#(从表的某个字段)的数据类型必须与主表名(被参考字段)的数据类型一致，逻辑意义也一样
#(从表的某个字段)的字段名可以与主表名(被参考字段)的字段名一样，也可以不一样

-- FOREIGN KEY: 在表级指定子表中的列
-- REFERENCES: 标示在父表中的列
```

<br/>

**‌**

```mysql
create table dept( #主表
	did int primary key,		#部门编号
	dname varchar(50)			#部门名称
);

create table emp(#从表
	eid int primary key,  #员工编号
	ename varchar(5),     #员工姓名
	deptid int,				#员工所在的部门
	foreign key (deptid) references dept(did)   #在从表中指定外键约束
	#emp表的deptid和和dept表的did的数据类型一致，意义都是表示部门的编号
);

说明：
（1）主表dept必须先创建成功，然后才能创建emp表，指定外键成功。
（2）删除表时，先删除从表emp，再删除主表dept
```

<br/><br/>
> <h3 id="建表后添加外键约束">建表后添加外键约束</h3>

一般情况下，表与表的关联都是提前设计好了的，因此，会在创建表的时候就把外键约束定义好。不过，如果需要修改表的设计（比如添加新的字段，增加新的关联关系），但没有预先定义外键约束，那么，就要用修改表的方式来补充定义。

<br/>

**格式：**

```mysql
ALTER TABLE 从表名 ADD [CONSTRAINT 约束名] FOREIGN KEY (从表的字段) REFERENCES 主表名(被引用字段) [on update xx][on delete xx];
```

举例：

```mysql
ALTER TABLE emp1
ADD [CONSTRAINT emp_dept_id_fk] FOREIGN KEY(dept_id) REFERENCES dept(dept_id);
```

<br/>

**举例：**

```mysql
create table dept(
	did int primary key,		#部门编号
	dname varchar(50)			#部门名称
);

create table emp(
	eid int primary key,  #员工编号
	ename varchar(5),     #员工姓名
	deptid int				#员工所在的部门
);
#这两个表创建时，没有指定外键的话，那么创建顺序是随意
```

```mysql
alter table emp add foreign key (deptid) references dept(did);
```

<br/><br/>
> <h3 id="外键-演示问题">外键-演示问题</h3>
<br/>

> <h3 id="失败：不是键列">失败：不是键列</h3>

```mysql
create table dept(
	did int ,		#部门编号
	dname varchar(50)			#部门名称
);

create table emp(
	eid int primary key,  #员工编号
	ename varchar(5),     #员工姓名
	deptid int,				#员工所在的部门
	foreign key (deptid) references dept(did)
);
#ERROR 1215 (HY000): Cannot add foreign key constraint  原因是dept的did不是键列
```

<br/><br/>
> <h3 id="demo-主表没有添加主键约束"> demo-主表没有添加主键约束</h3>

**DEMO:**

```sql
-- 先创建主表,再创建从表
-- 创建主表
CREATE TABLE dept1 (
 dept_id INT,
 dept_name VARCHAR(15)
);

-- 创建从表
CREATE TABLE emp1 (
	emp_id INT PRIMARY KEY AUTO_INCREMENT,
	emp_name VARCHAR(15),
	department_id INT,
	
	#表级约束: 
-- fk_emp1_dept_id: 定义为外键;
-- 	department_id: 从表emp1中的department_id列名,外键作用在department_id上
--  dept1: 主表名
--  dept_id: 主表的列名
	CONSTRAINT fk_emp1_dept_id FOREIGN KEY (department_id) REFERENCES dept1(dept_id)
);
```

报错了,如下:

```sh
ERROR 6125 (HY000): Failed to add the foreign key constraint. Missing unique key for constraint 'fk_emp1_dept_id' in the referenced table 'dept1'
```

**报错原因:** 主表dept1的dept_id没有添加主键约束或者唯一性约束.

```sql
ALTER TABLE dept1
ADD PRIMARY KEY (dept_id);


DESC dept1;
+-----------+-------------+------+-----+---------+-------+
| Field     | Type        | Null | Key | Default | Extra |
+-----------+-------------+------+-----+---------+-------+
| dept_id   | int         | NO   | PRI | NULL    |       |
| dept_name | varchar(15) | YES  |     | NULL    |       |
+-----------+-------------+------+-----+---------+-------+
2 rows in set (0.00 sec)
```

<br/>

**再来创建从表emp1:**

```sql
CREATE TABLE emp1 (
	emp_id INT PRIMARY KEY AUTO_INCREMENT,
	emp_name VARCHAR(15),
	department_id INT,
	
	#表级约束: 
-- fk_emp1_dept_id: 定义为外键;
-- 	department_id: 从表emp1中的department_id列名,外键作用在department_id上
--  dept1: 主表名
--  dept_id: 主表的列名
	CONSTRAINT fk_emp1_dept_id FOREIGN KEY (department_id) REFERENCES dept1(dept_id)
);

Query OK, 0 rows affected (0.01 sec)
```

<br/>

**查看从表emp1 结构:**

```sql
DESC emp1;
+---------------+-------------+------+-----+---------+----------------+
| Field         | Type        | Null | Key | Default | Extra          |
+---------------+-------------+------+-----+---------+----------------+
| emp_id        | int         | NO   | PRI | NULL    | auto_increment |
| emp_name      | varchar(15) | YES  |     | NULL    |                |
| department_id | int         | YES  | MUL | NULL    |                |
+---------------+-------------+------+-----+---------+----------------+
3 rows in set (0.01 sec)
```

<br/>

**查看约束结构:**

```sql
SELECT * FROM information_schema.table_constraints
WHERE table_name = 'emp1';

+--------------------+-------------------+-----------------+--------------+------------+-----------------+----------+
| CONSTRAINT_CATALOG | CONSTRAINT_SCHEMA | CONSTRAINT_NAME | TABLE_SCHEMA | TABLE_NAME | CONSTRAINT_TYPE | ENFORCED |
+--------------------+-------------------+-----------------+--------------+------------+-----------------+----------+
| def                | db_test           | PRIMARY         | db_test      | emp1       | PRIMARY KEY     | YES      |
| def                | db_test           | fk_emp1_dept_id | db_test      | emp1       | FOREIGN KEY     | YES      |
+--------------------+-------------------+-----------------+--------------+------------+-----------------+----------+
2 rows in set (0.01 sec)
```

<br/>

（2）失败：数据类型不一致

```mysql
create table dept(
	did int primary key,		#部门编号
	dname varchar(50)			#部门名称
);

create table emp(
	eid int primary key,  #员工编号
	ename varchar(5),     #员工姓名
	deptid char,				#员工所在的部门
	foreign key (deptid) references dept(did)
);
#ERROR 1215 (HY000): Cannot add foreign key constraint  原因是从表的deptid字段和主表的did字段的数据类型不一致，并且要它俩的逻辑意义一致
```

<br/>

（3）成功，两个表字段名一样

```mysql
create table dept(
	did int primary key,		#部门编号
	dname varchar(50)			#部门名称
);

create table emp(
	eid int primary key,  #员工编号
    ename varchar(5),     #员工姓名
    did int,				#员工所在的部门
    foreign key (did) references dept(did)  
    #emp表的deptid和和dept表的did的数据类型一致，意义都是表示部门的编号
    #是否重名没问题，因为两个did在不同的表中
);
```

<br/><br/>
> <h3 id="添加、删除、修改问题">添加、删除、修改问题</h3>

**建表:**

```mysql
create table dept(
	did int primary key,		#部门编号
	dname varchar(50)			#部门名称
);

create table emp(
	eid int primary key,  #员工编号
	ename varchar(5),     #员工姓名
	deptid int,				#员工所在的部门
	foreign key (deptid) references dept(did)  
	#emp表的deptid和和dept表的did的数据类型一致，意义都是表示部门的编号
);
```

<br/>

```mysql
insert into dept values(1001,'教学部');
insert into dept values(1003, '财务部');

insert into emp values(1,'张三',1001); #添加从表记录成功，在添加这条记录时，要求部门表有1001部门

insert into emp values(2,'李四',1005);#添加从表记录失败

ERROR 1452 (23000): Cannot add（添加） or update（修改） a child row: a foreign key constraint fails (`atguigudb`.`emp`, CONSTRAINT `emp_ibfk_1` FOREIGN KEY (`deptid`) REFERENCES `dept` (`did`)) 从表emp添加记录失败，因为主表dept没有1005部门
```

<br/>

```mysql
mysql> select * from dept;
+------+--------+
| did  | dname  |
+------+--------+
| 1001 | 教学部  |
| 1003 | 财务部  |
+------+--------+
2 rows in set (0.00 sec)

mysql> select * from emp;
+-----+-------+--------+
| eid | ename | deptid |
+-----+-------+--------+
|   1 | 张三   |   1001 |
+-----+-------+--------+
1 row in set (0.00 sec)
```

<br/>

```mysql
update emp set deptid = 1002 where eid = 1;#修改从表失败 
ERROR 1452 (23000): Cannot add（添加） or update（修改） a child row（子表的记录）: a foreign key constraint fails（外键约束失败） (`atguigudb`.`emp`, CONSTRAINT `emp_ibfk_1` FOREIGN KEY (`deptid`) REFERENCES `dept` (`did`))  #部门表did字段现在没有1002的值，所以员工表中不能修改员工所在部门deptid为1002

update dept set did = 1002 where did = 1001;#修改主表失败
ERROR 1451 (23000): Cannot delete（删除） or update（修改） a parent row（父表的记录）: a foreign key constraint fails (`atguigudb`.`emp`, CONSTRAINT `emp_ibfk_1` FOREIGN KEY (`deptid`) REFERENCES `dept` (`did`)) #部门表did的1001字段已经被emp引用了，所以部门表的1001字段就不能修改了。

update dept set did = 1002 where did = 1003;#修改主表成功  因为部门表的1003部门没有被emp表引用，所以可以修改
```

<br/>

```mysql
delete from dept where did=1001; #删除主表失败
ERROR 1451 (23000): Cannot delete（删除） or update（修改） a parent row（父表记录）: a foreign key constraint fails (`atguigudb`.`emp`, CONSTRAINT `emp_ibfk_1` FOREIGN KEY (`deptid`) REFERENCES `dept` (`did`))  #因为部门表did的1001字段已经被emp引用了，所以部门表的1001字段对应的记录就不能被删除
```


<br/><br/>
> <h3 id="demo-插入失败">demo-插入失败</h3>

之前创建的从表(员工表)emp1,添加失败: 

```sql
INSERT INTO emp1
VALUES(1001, 'Tom', 10);

ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails (`db_test`.`emp1`, CONSTRAINT `fk_emp1_dept_id` FOREIGN KEY (`department_id`) REFERENCES `dept1` (`dept_id`))
```

因为添加的部门id(department_id)10还没有加入主表,所以失败.

<br/>

**主表(部门表dept1)插入数据:** 添加了10号部门,然后就可以在从表中添加10号部门的员工了.

```sql
INSERT INTO dept1
VALUES(10, 'IT');

Query OK, 1 row affected (0.00 sec)
```

<br/>

再次添加员工就成功了:

```sql
INSERT INTO emp1
VALUES(1001, 'Tom', 10);

Query OK, 1 row affected (0.00 sec)
```

<br/><br/>
> <h3 id="demo-删除失败">demo-删除失败</h3>

```sql
DELETE FROM dept1
WHERE dept_id = 10;

ERROR 1451 (23000): Cannot delete or update a parent row: a foreign key constraint fails (`db_test`.`emp1`, CONSTRAINT `fk_emp1_dept_id` FOREIGN KEY (`department_id`) REFERENCES `dept1` (`dept_id`))
```

因为删除了主表中的数据,从表中的数据没有删除成了无根之萍了.删除的时候要先删除从表数据.

<br/><br/>
> <h3 id="demo-修改主表数据失败">demo-修改主表数据失败</h3>


修改10号部门为20号部门:

```sql
UPDATE dept1
set dept_id = 20
WHERE dept_id =10;

ERROR 1451 (23000): Cannot delete or update a parent row: a foreign key constraint fails (`db_test`.`emp1`, CONSTRAINT `fk_emp1_dept_id` FOREIGN KEY (`department_id`) REFERENCES `dept1` (`dept_id`))
```

更新失败了!!


***
<br/>

- **总结：约束关系是针对双方的**
	- 添加了外键约束后，主表的修改和删除数据受约束
	- 添加了外键约束后，从表的添加和修改数据受约束
	- 在从表上建立外键，要求主表必须存在
	- 删除主表时，要求从表从表先删除，或将从表中外键引用该主表的关系先删除

<br/><br/>
> <h3 id="约束等级">约束等级</h3>

* `Cascade方式`：在父表上update/delete记录时，同步update/delete掉子表的匹配记录 

* `Set null方式`：在父表上update/delete记录时，将子表上匹配记录的列设为null，但是要注意子表的外键列不能为not null  

* `No action方式`：如果子表中有匹配的记录，则不允许对父表对应候选键进行update/delete操作  

* `Restrict方式`：同no action， 都是立即检查外键约束

* `Set default方式`（在可视化工具SQLyog中可能显示空白）：父表有变更时，子表将外键列设置成一个默认的值，但Innodb不能识别

如果没有指定等级，就相当于Restrict方式。

对于外键约束，最好是采用: `ON UPDATE CASCADE ON DELETE RESTRICT` 的方式。

（1）演示1：on update cascade on delete set null

```mysql
create table dept(
	did int primary key,		#部门编号
	dname varchar(50)			#部门名称
);

create table emp(
	eid int primary key,  #员工编号
	ename varchar(5),     #员工姓名
	deptid int,				#员工所在的部门
	foreign key (deptid) references dept(did)  on update cascade on delete set null
	#把修改操作设置为级联修改等级，把删除操作设置为set null等级
);
```
<br/>

```mysql
insert into dept values(1001,'教学部');
insert into dept values(1002, '财务部');
insert into dept values(1003, '咨询部');


insert into emp values(1,'张三',1001); #在添加这条记录时，要求部门表有1001部门
insert into emp values(2,'李四',1001);
insert into emp values(3,'王五',1002);

```
<br/>

```mysql
mysql> select * from dept;

mysql> select * from emp;

```
<br/>

```mysql
#修改主表成功，从表也跟着修改，修改了主表被引用的字段1002为1004，从表的引用字段就跟着修改为1004了
mysql> update dept set did = 1004 where did = 1002;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from dept;
+------+--------+
| did  | dname  |
+------+--------+
| 1001 | 教学部 |
| 1003 | 咨询部 |
| 1004 | 财务部 | #原来是1002，修改为1004
+------+--------+
3 rows in set (0.00 sec)

mysql> select * from emp;
+-----+-------+--------+
| eid | ename | deptid |
+-----+-------+--------+
|   1 | 张三  |   1001 |
|   2 | 李四  |   1001 |
|   3 | 王五  |   1004 | #原来是1002，跟着修改为1004
+-----+-------+--------+
3 rows in set (0.00 sec)
```

<br/>

```mysql
#删除主表的记录成功，从表对应的字段的值被修改为null
mysql> delete from dept where did = 1001;
Query OK, 1 row affected (0.01 sec)

mysql> select * from dept;
+------+--------+
| did  | dname  | #记录1001部门被删除了
+------+--------+
| 1003 | 咨询部  |
| 1004 | 财务部  |
+------+--------+
2 rows in set (0.00 sec)

mysql> select * from emp;
+-----+-------+--------+
| eid | ename | deptid |
+-----+-------+--------+
|   1 | 张三  |   NULL | #原来引用1001部门的员工，deptid字段变为null
|   2 | 李四  |   NULL |
|   3 | 王五  |   1004 |
+-----+-------+--------+
3 rows in set (0.00 sec)
```

<br/>

（2）演示2：on update set null on delete cascade

```mysql
create table dept(
	did int primary key,		#部门编号
    dname varchar(50)			#部门名称
);

create table emp(
	eid int primary key,  #员工编号
    ename varchar(5),     #员工姓名
    deptid int,				#员工所在的部门
    foreign key (deptid) references dept(did)  on update set null on delete cascade
    #把修改操作设置为set null等级，把删除操作设置为级联删除等级
);
```

```mysql
insert into dept values(1001,'教学部');
insert into dept values(1002, '财务部');
insert into dept values(1003, '咨询部');

insert into emp values(1,'张三',1001); #在添加这条记录时，要求部门表有1001部门
insert into emp values(2,'李四',1001);
insert into emp values(3,'王五',1002);
```

```mysql
mysql> select * from dept;
+------+--------+
| did  | dname  |
+------+--------+
| 1001 | 教学部 |
| 1002 | 财务部 |
| 1003 | 咨询部 |
+------+--------+
3 rows in set (0.00 sec)

mysql> select * from emp;
+-----+-------+--------+
| eid | ename | deptid |
+-----+-------+--------+
|   1 | 张三  |   1001 |
|   2 | 李四  |   1001 |
|   3 | 王五  |   1002 |
+-----+-------+--------+
3 rows in set (0.00 sec)
```

```mysql
#修改主表，从表对应的字段设置为null
mysql> update dept set did = 1004 where did = 1002;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from dept;
+------+--------+
| did  | dname  |
+------+--------+
| 1001 | 教学部 |
| 1003 | 咨询部 |
| 1004 | 财务部 | #原来did是1002
+------+--------+
3 rows in set (0.00 sec)

mysql> select * from emp;
+-----+-------+--------+
| eid | ename | deptid |
+-----+-------+--------+
|   1 | 张三  |   1001 |
|   2 | 李四  |   1001 |
|   3 | 王五  |   NULL | #原来deptid是1002，因为部门表1002被修改了，1002没有对应的了，就设置为null
+-----+-------+--------+
3 rows in set (0.00 sec)
```

```mysql
#删除主表的记录成功，主表的1001行被删除了，从表相应的记录也被删除了
mysql> delete from dept where did=1001;
Query OK, 1 row affected (0.00 sec)

mysql> select * from dept;
+------+--------+
| did  | dname  | #部门表中1001部门被删除
+------+--------+
| 1003 | 咨询部 |
| 1004 | 财务部 |
+------+--------+
2 rows in set (0.00 sec)

mysql> select * from emp;
+-----+-------+--------+
| eid | ename | deptid |#原来1001部门的员工也被删除了
+-----+-------+--------+
|   3 | 王五  |   NULL |
+-----+-------+--------+
1 row in set (0.00 sec)

```

<br/>

（3）演示：on update cascade on delete cascade

```mysql
create table dept(
	did int primary key,		#部门编号
    dname varchar(50)			#部门名称
);

create table emp(
	eid int primary key,  #员工编号
    ename varchar(5),     #员工姓名
    deptid int,				#员工所在的部门
    foreign key (deptid) references dept(did)  on update cascade on delete cascade
    #把修改操作设置为级联修改等级，把删除操作也设置为级联删除等级
);
```

```mysql
insert into dept values(1001,'教学部');
insert into dept values(1002, '财务部');
insert into dept values(1003, '咨询部');

insert into emp values(1,'张三',1001); #在添加这条记录时，要求部门表有1001部门
insert into emp values(2,'李四',1001);
insert into emp values(3,'王五',1002);
```

```mysql
mysql> select * from dept;
+------+--------+
| did  | dname  |
+------+--------+
| 1001 | 教学部 |
| 1002 | 财务部 |
| 1003 | 咨询部 |
+------+--------+
3 rows in set (0.00 sec)

mysql> select * from emp;
+-----+-------+--------+
| eid | ename | deptid |
+-----+-------+--------+
|   1 | 张三  |   1001 |
|   2 | 李四  |   1001 |
|   3 | 王五  |   1002 |
+-----+-------+--------+
3 rows in set (0.00 sec)
```

```mysql
#修改主表，从表对应的字段自动修改
mysql> update dept set did = 1004 where did = 1002;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from dept;
+------+--------+
| did  | dname  |
+------+--------+
| 1001 | 教学部 |
| 1003 | 咨询部 |
| 1004 | 财务部 | #部门1002修改为1004
+------+--------+
3 rows in set (0.00 sec)

mysql> select * from emp;
+-----+-------+--------+
| eid | ename | deptid |
+-----+-------+--------+
|   1 | 张三  |   1001 |
|   2 | 李四  |   1001 |
|   3 | 王五  |   1004 | #级联修改
+-----+-------+--------+
3 rows in set (0.00 sec)
```

```mysql
#删除主表的记录成功，主表的1001行被删除了，从表相应的记录也被删除了
mysql> delete from dept where did=1001;
Query OK, 1 row affected (0.00 sec)

mysql> select * from dept;
+------+--------+
| did  | dname  | #1001部门被删除了
+------+--------+
| 1003 | 咨询部 |
| 1004 | 财务部 | 
+------+--------+
2 rows in set (0.00 sec)

mysql> select * from emp;
+-----+-------+--------+
| eid | ename | deptid |  #1001部门的员工也被删除了
+-----+-------+--------+
|   3 | 王五  |   1004 |
+-----+-------+--------+
1 row in set (0.00 sec)
```



<br/><br/>
> <h3 id="删除外键约束">删除外键约束</h3>
流程如下：

```mysql
(1)第一步先查看约束名和删除外键约束
SELECT * FROM information_schema.table_constraints WHERE table_name = '表名称';#查看某个表的约束名

ALTER TABLE 从表名 DROP FOREIGN KEY 外键约束名;

（2）第二步查看索引名和删除索引。（注意，只能手动删除）
SHOW INDEX FROM 表名称; #查看某个表的索引名

ALTER TABLE 从表名 DROP INDEX 索引名;

```

<br/>

举例：

```mysql
mysql> SELECT * FROM information_schema.table_constraints WHERE table_name = 'emp';

mysql> alter table emp drop foreign key emp_ibfk_1;
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

```

```mysql
mysql> show index from emp;

mysql> alter table emp drop index deptid;
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql>  show index from emp;
```

<br/><br/>
> <h3 id="外键-开发场景"> 外键-开发场景 </h3>

**问题1：如果两个表之间有关系（一对一、一对多），比如：员工表和部门表（一对多），它们之间是否一定要建外键约束？**

答：不是的

<br/>

**问题2：建和不建外键约束有什么区别？**

答：建外键约束，你的操作（创建表、删除表、添加、修改、删除）会受到限制，从语法层面受到限制。例如：在员工表中不可能添加一个员工信息，它的部门的值在部门表中找不到。

不建外键约束，你的操作（创建表、删除表、添加、修改、删除）不受限制，要保证数据的`引用完整性`，只能依`靠程序员的自觉`，或者是`在Java程序中进行限定`。例如：在员工表中，可以添加一个员工的信息，它的部门指定为一个完全不存在的部门。

<br/>

**问题3：那么建和不建外键约束和查询有没有关系？**

答：没有

> 在 MySQL 里，外键约束是有成本的，需要消耗系统资源。对于大并发的 SQL 操作，有可能会不适合。比如大型网站的中央数据库，可能会`因为外键约束的系统开销而变得非常慢`。所以， MySQL 允许你不使用系统自带的外键约束，在`应用层面`完成检查数据一致性的逻辑。也就是说，即使你不用外键约束，也要想办法通过应用层面的附加逻辑，来实现外键约束的功能，确保数据的一致性。

<br/>

**阿里开发规范**

【`强制`】不得使用外键与级联，一切外键概念必须在应用层解决。 

说明：（概念解释）学生表中的 student_id 是主键，那么成绩表中的 student_id 则为外键。如果更新学生表中的 student_id，同时触发成绩表中的 student_id 更新，即为级联更新。外键与级联更新适用于`单机低并发`，不适合`分布式`、`高并发集群`；级联更新是强阻塞，存在数据库`更新风暴`的风险；外键影响数据库的`插入速度`。





***
<br/><br/><br/>

> <h2 id="检查约束check">检查约束check</h2>
**MySQL 5.7 不支持**

MySQL5.7 可以使用check约束，但check约束对数据验证没有任何作用。添加数据时，没有任何错误或警告

<br/>
但是MySQL 8.0中可以使用check约束了。

```sql
create table employee(
	eid int primary key,
	ename varchar(5),
	gender char check ('男' or '女')
);

insert into employee values(1,'张三','妖');


mysql> select * from employee;
+-----+-------+--------+
| eid | ename | gender |
+-----+-------+--------+
|   1 | 张三   | 妖     |
+-----+-------+--------+
1 row in set (0.00 sec)
```


***
<br/><br/><br/>

> <h2 id="默认值约束">默认值约束</h2>
**作用:** 给某个字段/某列指定默认值，一旦设置默认值，在插入数据时，如果此字段没有显式赋值，则赋值为默认值。

<br/>

```sql
create table 表名称(
	字段名  数据类型  primary key,
	字段名  数据类型  unique key not null,  
	字段名  数据类型  unique key,
	字段名  数据类型  not null default 默认值, 
);

create table 表名称(
	字段名  数据类型 default 默认值 ,
	字段名  数据类型 not null default 默认值,  
	字段名  数据类型 not null default 默认值,
	primary key(字段名),
	unique key(字段名)
);

说明：默认值约束一般不在唯一键和主键列上加
```

<br/>

```sql
create table employee(
	eid int primary key,
	ename varchar(20) not null,
	gender char default '男',
	tel char(11) not null default '' #默认是空字符串
);
```

<br/>

```sql
desc employee;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| eid    | int(11)     | NO   | PRI | NULL    |       |
| ename  | varchar(20) | NO   |     | NULL    |       |
| gender | char(1)     | YES  |     | 男      |       |
| tel    | char(11)    | NO   |     |         |       |
+--------+-------------+------+-----+---------+-------+
4 rows in set (0.00 sec)
```


<br/>

```sql
insert into employee values(1,'汪飞','男','13700102535'); #成功

mysql> select * from employee;
+-----+-------+--------+-------------+
| eid | ename | gender | tel         |
+-----+-------+--------+-------------+
|   1 | 汪飞  | 男     | 13700102535 |
+-----+-------+--------+-------------+
1 row in set (0.00 sec)
```

<br/>

```sql
insert into employee(eid,ename) values(2,'天琪'); #成功

mysql> select * from employee;
+-----+-------+--------+-------------+
| eid | ename | gender | tel         |
+-----+-------+--------+-------------+
|   1 | 汪飞  | 男     | 13700102535 |
|   2 | 天琪  | 男     |             |
+-----+-------+--------+-------------+
2 rows in set (0.00 sec)
```

***
<br/><br/><br/>

> <h2 id="添加/删除约束">添加/删除约束</h2>

`CREATE TABLE`添加约束

`ALTER TABLE`添加约束、删除约束

<br/><br/>
> <h3 id="查看表中所有约束">查看表中所有约束</h3>

```sql
SELECT * FROM information_schema.table_constraints WHERE table_name = '表名'; #查看都有哪些约束
```

<br/>

**举例:**

```sql
SELECT * FROM information_schema.table_constraints WHERE table_name = 'employees'; #查看都有哪些约束

+--------------------+-------------------+-----------------+--------------+------------+-----------------+----------+
| CONSTRAINT_CATALOG | CONSTRAINT_SCHEMA | CONSTRAINT_NAME | TABLE_SCHEMA | TABLE_NAME | CONSTRAINT_TYPE | ENFORCED |
+--------------------+-------------------+-----------------+--------------+------------+-----------------+----------+
| def                | db_test           | idx_email       | db_test      | employees  | UNIQUE          | YES      |
| def                | db_test           | PRIMARY         | db_test      | employees  | PRIMARY KEY     | YES      |
+--------------------+-------------------+-----------------+--------------+------------+-----------------+----------+
2 rows in set (0.01 sec)
```


***
<br/><br/><br/>

> <h2 id="约束-思考题"> 约束-思考题 </h2>

**1、为什么建表时，加 not null default '' 或 default 0**

答：不想让表中出现null值。

<br/>

**2、为什么不想要 null 的值**

答:（1）不好比较。null是一种特殊值，比较时只能用专门的is null 和 is not null来比较。碰到运算符，通常返回null。

​     （2）效率不高。影响提高索引效果。因此，我们往往在建表时 not null default '' 或 default 0

<br/>

**3、带AUTO_INCREMENT约束的字段值是从1开始的吗？**
在MySQL中，默认AUTO_INCREMENT的初始值是1，每新增一条记录，字段值自动加1。设置自增属性（AUTO_INCREMENT）的时候，还可以指定第一条插入记录的自增字段的值，这样新插入的记录的自增字段值从初始值开始递增，如在表中插入第一条记录，同时指定id值为5，则以后插入的记录的id值就会从6开始往上增加。添加主键约束时，往往需要设置字段自动增加属性。

<br/>

**4、并不是每个表都可以任意选择存储引擎？**
外键约束（FOREIGN KEY）不能跨引擎使用。

MySQL支持多种存储引擎，每一个表都可以指定一个不同的存储引擎，需要注意的是：外键约束是用来保证数据的参照完整性的，如果表之间需要关联外键，却指定了不同的存储引擎，那么这些表之间是不能创建外键约束的。所以说，存储引擎的选择也不完全是随意的。


<br/><br/><br/>

***
<br/>

> <h1 id="视图">视图</h1>

**为什么使用视图？**

视图一方面可以帮我们使用表的一部分而不是所有的表，另一方面也可以针对不同的用户制定不同的查询视图。比如，针对一个公司的销售人员，我们只想给他看部分数据，而某些特殊的数据，比如采购的价格，则不会提供给他。再比如，人员薪酬是个敏感的字段，那么只给某个级别以上的人员开放，其他人的查询视图中则不提供这个字段。

刚才讲的只是视图的一个使用场景，实际上视图还有很多作用。最后，我们总结视图的优点。

<br/>

![go.0.0.129.png](./../Pictures/go.0.0.129.png)

<br/>

![go.0.0.130.png](./../Pictures/go.0.0.130.png)

- **视图理解:**
	- 视图,可以看作是一个虚拟表,本身是不存储数据的.视图的本质,就可以看作是存储起来的SELECT语句.
	- 视图中的SELECT语句中涉及到的表,称为基表;
	- 针对视图做DML操作,会影响到对应的基表中的数据.反之亦然.
	- 视图本身的删除,不会导致基表中数据的删除.
	- 视图的应用场景:针对小型项目,不推荐使用视图.针对大型项目,可以考虑使用视图.
	- 视图优点:简化查询;控制数据的访问.

***
<br/><br/><br/>
> <h2 id="创建视图">创建视图</h2>

**CREATE VIEW 语句中嵌入子查询**

```sql
CREATE [OR REPLACE] 
[ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}] 
VIEW 视图名称 [(字段列表)]
AS 查询语句
[WITH [CASCADED|LOCAL] CHECK OPTION]
```

***
<br/><br/><br/>
> <h2 id="创建单表视图">创建单表视图</h2>
**精简版**

```sql
CREATE VIEW 视图名称 
AS 查询语句
```

<br/>

```sql
-- 拷贝的时候,约束、外键没有拷贝过来,只是数据拷贝过来和非空
-- 拷贝一份employees表,作为视图的测试表
CREATE TABLE emps
AS
SELECT *
FROM employees


CREATE TABLE depts
AS
SELECT *
FROM departments;
```

<br/>

```sql
CREATE VIEW empvu80
AS 
SELECT  employee_id, last_name, salary
FROM    employees
WHERE   department_id = 80;

SELECT *
FROM	salvu80;
```

![go.0.0.131.png](./../Pictures/go.0.0.131.png)

<br/>

**DEMO1**

```sql
-- 固定下来作为视图 emp_year_salary
CREATE VIEW emp_year_salary (ename,year_salary)
AS 
SELECT ename,salary*12*(1+IFNULL(commission_pct,0))
FROM t_employee;
```

<br/>

**DEMO2**

```sql
-- 固定下来作为视图 salvu50
CREATE VIEW salvu50
AS 
SELECT  employee_id ID_NUMBER, last_name NAME,salary*12 ANN_SALARY
FROM    employees
WHERE   department_id = 50;
```

**说明1：** 实际上就是我们在 SQL 查询语句的基础上封装了视图 VIEW，这样就会基于 SQL 语句的结果集形成一张虚拟表。

**说明2 ：** 在创建视图时，没有在视图名后面指定字段列表，则视图中字段列表默认和SELECT语句中的字段列表一致。如果SELECT语句中给字段取了别名，那么视图中的字段名和别名相同。

***
<br/><br/><br/>
> <h2 id="查看视图">查看视图</h2>

**语法1：查看数据库的表对象、视图对象**

```mysql
SHOW TABLES;
```

<br/>

**语法2：查看视图的结构**

```mysql
DESC / DESCRIBE 视图名称;
```

<br/>

**语法3：查看视图的属性信息**

```mysql
# 查看视图信息（显示数据表的存储引擎、版本、数据行数和数据大小等）
SHOW TABLE STATUS LIKE '视图名称'\G
```

<br/>

执行结果显示，注释Comment为VIEW，说明该表为视图，其他的信息为NULL，说明这是一个虚表。

**语法4：查看视图的详细定义信息**

```mysql
SHOW CREATE VIEW 视图名称;
```


***
<br/><br/><br/>

> <h2 id="更新视图的数据">更新视图的数据</h2>
MySQL支持使用INSERT、UPDATE和DELETE语句对视图中的数据进行插入、更新和删除操作。当视图中的数据发生变化时，数据表中的数据也会发生变化，反之亦然。

**UPDATE操作:**

```sql
mysql> SELECT ename,tel FROM emp_tel WHERE ename = '孙洪亮';
+---------+-------------+
| ename   | tel         |
+---------+-------------+
| 孙洪亮 	| 13789098765 |
+---------+-------------+
1 row in set (0.01 sec)

mysql> UPDATE emp_tel SET tel = '13789091234' WHERE ename = '孙洪亮';
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> SELECT ename,tel FROM emp_tel WHERE ename = '孙洪亮';
+---------+-------------+
| ename	  | tel         |
+---------+-------------+
| 	孙洪亮 | 13789091234 |
+---------+-------------+
1 row in set (0.00 sec)

mysql> SELECT ename,tel FROM t_employee WHERE ename = '孙洪亮';
+---------+-------------+
| ename   | tel         |
+---------+-------------+
| 孙洪亮 	| 13789091234 |
+---------+-------------+
1 row in set (0.00 sec)
```

***
<br/><br/><br/>

> <h2 id="不可更新的视图">不可更新的视图</h2>
要使视图可更新，视图中的行和底层基本表中的行之间必须存在`一对一`的关系。另外当视图定义出现如下情况时，视图不支持更新操作：

- 在定义视图的时候指定了“ALGORITHM = TEMPTABLE”，视图将不支持INSERT和DELETE操作；
- 视图中不包含基表中所有被定义为非空又未指定默认值的列，视图将不支持INSERT操作；
- 在定义视图的SELECT语句中使用了`JOIN联合查询`，视图将不支持INSERT和DELETE操作；
- 在定义视图的SELECT语句后的字段列表中使用了`数学表达式`或`子查询`，视图将不支持INSERT，也不支持UPDATE使用了数学表达式、子查询的字段值；
- 在定义视图的SELECT语句后的字段列表中使用`DISTINCT`、`聚合函数`、`GROUP BY`、`HAVING`、`UNION`等，视图将不支持INSERT、UPDATE、DELETE；
- 在定义视图的SELECT语句中包含了子查询，而子查询中引用了FROM后面的表，视图将不支持INSERT、UPDATE、DELETE；
- 视图定义基于一个`不可更新视图`；
- 常量视图。

<br/>

举例：

```mysql
mysql> CREATE OR REPLACE VIEW emp_dept
    -> (ename,salary,birthday,tel,email,hiredate,dname)
    -> AS SELECT ename,salary,birthday,tel,email,hiredate,dname
    -> FROM t_employee INNER JOIN t_department
    -> ON t_employee.did = t_department.did ;
Query OK, 0 rows affected (0.01 sec)

```

```mysql
mysql> INSERT INTO emp_dept(ename,salary,birthday,tel,email,hiredate,dname)
    -> VALUES('张三',15000,'1995-01-08','18201587896',
    -> 'zs@atguigu.com','2022-02-14','新部门');
    
#ERROR 1393 (HY000): Can not modify more than one base table through a join view 'atguigu_chapter9.emp_dept'
```

从上面的SQL执行结果可以看出，在定义视图的SELECT语句中使用了JOIN联合查询，视图将不支持更新操作。

> 虽然可以更新视图数据，但总的来说，视图作为`虚拟表`，主要用于`方便查询`，不建议更新视图的数据。**对视图数据的更改，都是通过对实际数据表里数据的操作来完成的。**