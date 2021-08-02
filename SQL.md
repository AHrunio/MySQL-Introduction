[TOC]

# SQL语言

## SQL基本语法

### 数据类型

![image-20210704151935385](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704151935385.png)

![image-20210704152004271](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704152004271.png)

PS：时间戳从1970.1.1 00:00到现在的毫 秒 

⭐下文中<>表示选择其一 ，[]表示可选可不选
⭐SQL语言的兼容大小写

### 数据库字段属性⭐

```mysql
unsigned -- 无符号整数声明该列不能为负
zerofill -- 0填充 int(3), 5 -> 005
auto_increment -- 自增 在上一条记录的基础上+1 必须是int 可以设定起始值和步长
not null -- 非空 不赋值就会报错
default  --  默认值
```

### 数据表类型（引擎）

默认使用InnoDB：安全性高，事务处理，多表多用户操作

早年使用MYISAM：节约空间，速度快

|            | MYISAM | InnoDB          |
| ---------- | ------ | --------------- |
| 事物支持   | N      | Y               |
| 数据行锁定 | N      | Y               |
| 外键约束   | N      | Y               |
| 全文索引   | Y      | Y（只支持英文） |
| 表空间大小 | 小     | 打              |

```mysql
-- 查看mysql所支持的引擎类型 (表类型)
SHOW ENGINES;
```



### 连接数据库

```mysql
show databases; 	--查看所有数据库
use dbname; 		--切换到dbname数据库
show tables; 		--查看数据库中所有的表
describe tbname; 	--显示tbname表的所有信息
create database dbname; --创建数据库dbname
show create database; --查看创建数据库的语句
```



### 模式的定义&删除

#### 模式定义

模式

```mysql
CREATE SCHEMA<模式名>AUTHORIZATION<用户名>;
```

模式+视图

```mysql
CREATE SCHEMA<模式名>AUTHORIZATION<用户名>[<表定义子句>|<视图定义子句>|<授权定义子句>];
```

例：

```mysql
create schema "LEARN" authorization Ahrun create table user(
id int primary key,age int ,name varchar(255)
);
```

#### 模式删除

```mysql
DROP SCHEMA<模式名><CASCADE|RESTRICT>;
```

删除模式，CASCADE&RESTRICT**必须**二选一

<img src="https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704153847922.png" alt="image-20210704153847922" style="zoom:150%;" />

### 表的定义&删除&修改

```mysql
show create table; --查看创建表的语句
desc tbname; --显示tbname表的结构
```

#### 表定义

```mysql
create table 表名(字段名 类型 字段约束，字段名 类型 字段约束，字段名 类型 字段约束，...);
```

例：

```mysql
create table user(
name varchar(10),
age int,
gender varchar(6)
);
```

![image-20210708162452874](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210708162452874.png)

![image-20210708162243573](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210708162243573.png)

#### 表删除

```mysql
DROP TABLE [IF EXISTS] <表名>[CASCADE|RESTRICT];
```

例：

```mysql
drop table user CASCADE;
```

<img src="https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704154733606.png" alt="image-20210704154733606" style="zoom:150%;" />

外键 ： 两表的码键（字段名）关联，

#### 表修改

```mysql
修改表名 :
ALTER TABLE 旧表名 RENAME AS 新表名
添加字段 : 
ALTER TABLE 表名 ADD字段名 列属性[属性]
修改字段 :
- ALTER TABLE 表名 MODIFY 字段名 列类型[属性]
- ALTER TABLE 表名 CHANGE 旧字段名 新字段名 列属性[属性]
删除字段 :  ALTER TABLE 表名 DROP 字段名
```



```mysql
ALTER TABLE<表名>
[ADD[COLUMN]<新列名><数据类型>[完整性约束]]
[ADD<表级完整性约束>]
```

### 外键

如果公共关键字在一个关系中是主关键字，那么这个公共关键字被称为另一个关系的外键。由此可见，外键表示了两个关系之间的相关联系。以另一个关系的外键作主关键字的表被称为**主表**，具有此外键的表被称为主表的**从表**。

#### 外键建立

建表时指定外键约束

```mysql
-- 创建外键的方式一 : 创建子表同时创建外键

-- 年级表 (id\年级名称)
CREATE TABLE `grade` (
`gradeid` INT(10) NOT NULL AUTO_INCREMENT COMMENT '年级ID',
`gradename` VARCHAR(50) NOT NULL COMMENT '年级名称',
PRIMARY KEY (`gradeid`)
) ENGINE=INNODB DEFAULT CHARSET=utf8

-- 学生信息表 (学号,姓名,性别,年级,手机,地址,出生日期,邮箱,身份证号)
CREATE TABLE `student` (
`studentno` INT(4) NOT NULL COMMENT '学号',
`studentname` VARCHAR(20) NOT NULL DEFAULT '匿名' COMMENT '姓名',
`gradeid` INT(10) DEFAULT NULL COMMENT '年级',
PRIMARY KEY (`studentno`),
KEY `FK_gradeid` (`gradeid`),-- 索引
CONSTRAINT `FK_gradeid` FOREIGN KEY (`gradeid`) REFERENCES `grade` (`gradeid`)
) ENGINE=INNODB DEFAULT CHARSET=utf8
```

建表后修改

```mysql
-- 创建外键方式二 : 创建子表完毕后,修改子表添加外键
ALTER TABLE `student`
ADD CONSTRAINT `FK_gradeid` FOREIGN KEY (`gradeid`) REFERENCES `grade` (`gradeid`);
```

#### 外键删除

操作：删除 grade 表，发现报错

![image-20210708172449345](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210708172449345.png)

**注意** : 删除具有主外键关系的表时 , 要先删子表 , 后删主表

```mysql
-- 删除外键
ALTER TABLE student DROP FOREIGN KEY FK_gradeid;
-- 注:这个索引是建立外键的时候生成的
-- 发现执行完上面的,索引还在,所以还要删除索引
ALTER TABLE student DROP INDEX FK_gradeid;
```

### 索引的建立h&删除&修改

请阅读：http://blog.codinglabs.org/articles/theory-of-mysql-index.html

数据量大时，查询消耗时间长，建立索引可以有效减少消耗时间。索引可以建立在一列或多列上。

- 使用分组和排序子句进行数据检索时 , 可以显著减少分组和排序的时间
- 全文检索字段进行搜索优化.

#### 索引分类

- 主键索引 (Primary Key) : 某一个属性组能唯一标识一条记录

  - 最常见的索引类型
  - 确保数据记录的唯一性
  - 确定特定数据记录在数据库中的位置

- 唯一索引 (Unique) : 避免同一个表中某数据列中的值重复：

  - 主键索引只能有一个
  - 唯一索引可能有多个

  ```mysql
  CREATE TABLE `Grade`(
    `GradeID` INT(11) AUTO_INCREMENT PRIMARYKEY,
    `GradeName` VARCHAR(32) NOT NULL UNIQUE
     -- 或 UNIQUE KEY `GradeID` (`GradeID`)
  )
  ```

- 常规索引 (Index): 快速定位特定数据 :

  - index 和 key 关键字都可以设置常规索引
  - 应加在查询找条件的字段
  - 不宜添加太多常规索引,影响数据的插入,删除和修改操作

  ```mysql
  CREATE TABLE `result`(
     -- 省略一些代码
    INDEX/KEY `ind` (`studentNo`,`subjectNo`) -- 创建表时添加
  )
  -- 创建后添加
  ALTER TABLE `result` ADD INDEX `ind`(`studentNo`,`subjectNo`);
  ```

- 全文索引 (FullText): 快速定位特定数据 

  - 只能用于CHAR , VARCHAR , TEXT数据列类型
  - 适合大型数据集

+ 组合索引

#### 索引建立

```mysql
CREATE [UNIQUE][CLUSTER]INDEX<索引名>
ON<表名>(<列名>[<次序>][,<列名>[<次序>]]...);
```

<img src="https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704155758405.png" alt="image-20210704155758405" style="zoom:150%;" />

聚簇：A->B->C...

唯一:

```mysql
CREATE UNIQUE INDEX stu_id_idx ON user(id);'id设为索引且为唯一索引'
```

```mysql
/*
#方法一：创建表时
  　　CREATE TABLE 表名 (
               字段名1 数据类型 [完整性约束条件…],
               字段名2 数据类型 [完整性约束条件…],
               [UNIQUE | FULLTEXT | SPATIAL ]   INDEX | KEY
               [索引名] (字段名[(长度)] [ASC |DESC])
               );


#方法二：CREATE在已存在的表上创建索引
       CREATE [UNIQUE | FULLTEXT | SPATIAL ] INDEX 索引名
                    ON 表名 (字段名[(长度)] [ASC |DESC]) ;


#方法三：ALTER TABLE在已存在的表上创建索引
       ALTER TABLE 表名 ADD [UNIQUE | FULLTEXT | SPATIAL ] INDEX
                            索引名 (字段名[(长度)] [ASC |DESC]) ;
                           
                           
#删除索引：DROP INDEX 索引名 ON 表名字;
#删除主键索引: ALTER TABLE 表名 DROP PRIMARY KEY;


#显示索引信息: SHOW INDEX FROM student;
*/

/*增加全文索引*/
ALTER TABLE `school`.`student` ADD FULLTEXT INDEX `studentname` (`StudentName`);

/*EXPLAIN : 分析SQL语句执行性能*/
EXPLAIN SELECT * FROM student WHERE studentno='1000';

/*使用全文索引*/
-- 全文搜索通过 MATCH() 函数完成。
-- 搜索字符串作为 against() 的参数被给定。搜索以忽略字母大小写的方式执行。对于表中的每个记录行，MATCH() 返回一个相关性值。即，在搜索字符串与记录行在 MATCH() 列表中指定的列的文本之间的相似性尺度。
EXPLAIN SELECT *FROM student WHERE MATCH(studentname) AGAINST('love');
```



#### 索引修改

```mysql
ALTER INDEX<旧索引名> RENAME TO<新索引名>
'例'
ALTER INDEX stu_id_idx RENAME TO Sid;
```

#### 索引删除

```mysql
DROP INDEX<索引名>
'例'
DROP INDEX Sid;
```

#### 索引准则

- 索引不是越多越好
- 不要对经常变动的数据加索引
- 小数据量的表建议不要加索引
- 索引一般应加在查找条件的字段

#### 索引的数据结构

```
-- 我们可以在创建上述索引的时候，为其指定索引类型，分两类
hash类型的索引：查询单条快，范围查询慢
btree类型的索引：b+树，层数越多，数据量指数级增长（我们就用它，因为innodb默认支持它）

-- 不同的存储引擎支持的索引类型也不一样
InnoDB 支持事务，支持行级别锁定，支持 B-tree、Full-text 等索引，不支持 Hash 索引；
MyISAM 不支持事务，支持表级别锁定，支持 B-tree、Full-text 等索引，不支持 Hash 索引；
Memory 不支持事务，支持表级别锁定，支持 B-tree、Hash 等索引，不支持 Full-text 索引；
NDB 支持事务，支持行级别锁定，支持 Hash 索引，不支持 B-tree、Full-text 等索引；
Archive 不支持事务，支持表级别锁定，不支持 B-tree、Hash、Full-text 等索引；
```

### 查询⭐

> SELECT语法

```mysql
SELECT [ALL | DISTINCT]
{* | table.* | [table.field1[as alias1][,table.field2[as alias2]][,...]]}
FROM table_name [as table_alias]
  [left | right | inner join table_name2]  -- 联合查询
  [WHERE ...]  -- 指定结果需满足的条件
  [GROUP BY ...]  -- 指定结果按照哪几个字段来分组
  [HAVING]  -- 过滤分组的记录必须满足的次要条件
  [ORDER BY ...]  -- 指定查询记录按一个或多个条件排序
  [LIMIT {[offset,]row_count | row_countOFFSET offset}];
   -- 指定查询的记录从哪条至哪条
```

应用场景 :

- SELECT语句返回结果列中使用

- SELECT语句中的ORDER BY , HAVING等子句中使用

- DML语句中的 where 条件语句中使用表达式

  ```mysql
  -- selcet查询中可以使用表达式
  SELECT @@auto_increment_increment; -- 查询自增步长
  SELECT VERSION(); -- 查询版本号
  SELECT 100*3-1 AS 计算结果; -- 表达式
  
  -- 此操作后学员考试成绩集体提分一分，并查看
  SELECT studentno,StudentResult+1 AS '提分后' FROM result;
  ```

**数据查询：**

student：

![image-20210704164431234](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704164431234.png)

sb：

![image-20210704172455412](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704172455412.png)

```mysql
SELECT *
FROM user;
'等价于'
SELECT id,number,name,gender,createDt
FROM user;
```

![image-20210704161923959](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704161923959.png)                         		![image-20210704161930251](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704161930251.png)

**取别名**

![image-20210704161958638](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704161958638.png)			![image-20210704162011755](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704162011755.png)

![image-20210704162158668](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704162158668.png)或者+as

**CONCAT()函数拼接字符串**

```mysql
-- CONCAT()函数拼接字符串
SELECT CONCAT('姓名:',studentname) AS 新姓名 FROM student;
```

![image-20210708201124345](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210708201124345.png)

#### 结果去重

+DISTINCT

```mysql
SELECT DISTINCT <字段名> FROM <表名>;
```

![image-20210704162825723](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704162825723.png)					![image-20210704162838786](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704162838786.png)

#### 查询结果加条件

```mysql
SELECT <字段名> FROM <表名> WHERE<查询条件>;
```

![image-20210704163255330](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704163255330.png)

①比较

![image-20210704164552930](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704164552930.png)

![image-20210704164602893](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704164602893.png)

②范围

![image-20210704165457012](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704165457012.png)

![image-20210704165505883](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704165505883.png)

③字符匹配

![image-20210704165621247](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704165621247.png)

![image-20210704165628178](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704165628178.png)

字符串匹配时 ，**’%‘代表任意多个任意字符，’_‘代表任意一个任意字符**。

④select 1 in (1,2); 返回的是true

#### 模糊查询

![image-20210708204122716](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210708204122716.png)

#### 聚集函数

![image-20210704170551750](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704170551750.png)

①元组个数

![image-20210704170827980](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704170827980.png)

![image-20210704170838721](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704170838721.png)

②总和

![image-20210704170922106](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704170922106.png)

![image-20210704170946426](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704170946426.png)

不重复的值的和（+distinct）：

![image-20210704171109659](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704171109659.png)

![image-20210704171120997](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704171120997.png)

#### 分组查询

GROUP BY 分组，HAVING 筛选

```mysql
SELECT <字段名>FROM<表名>GROUP BY<字段名>[HAVING <约束>]
```

①筛选

![image-20210704171821875](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704171821875.png)

![image-20210704171838180](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704171838180.png)

![image-20210704171553426](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704171553426.png)

![image-20210704171602943](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704171602943.png)

②不加筛选条件：

![image-20210704171640666](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704171640666.png)

![image-20210704171617746](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704171617746.png)

### 连接

#### 等值&非等值连接

等值，非等值连接，以WHERE为关键词

![image-20210704172626796](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704172626796.png)

![image-20210704172413357](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704172610639.png)

#### 自身连接

一个表与自身连接，为本身这个表起两个别名，然后进行操作

![image-20210704172840648](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704172840648.png)

![image-20210704172848390](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704172848390.png)

#### 外连接

把被舍弃的值也保留在结果中，但是要加NULL

①左连接

![image-20210704173337800](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704173337800.png)

![image-20210704173350868](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704173350868.png)

②右连接

![image-20210704173502154](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704173502154.png)

![image-20210704173512220](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704173512220.png)



#### 内连接

INNER可省略

![image-20210704174112774](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704174112774.png)

![image-20210704174125970](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704174125970.png)

#### 多表查询

```mysql
SELECT * FROM SB1,SB2,SB3 WHERE SB1.id=SB2.id AND SB2.id=SB3.id
'等价于'
SELECT * FROM SB1 LEFT OUTER JOIN SB2 ON SB1.id=SB2.id LEFT OUTER JOIN SB3 ON SB2.id=SB3.id;
```

### 排序和分页

#### 排序

```mysql
/*============== 排序 ================
语法 : ORDER BY
   ORDER BY 语句用于根据指定的列对结果集进行排序。
   ORDER BY 语句默认按照ASC升序对记录进行排序。
   如果您希望按照降序对记录进行排序，可以使用 DESC 关键字。
*/
-- 查询 数据库结构-1 的所有考试结果(学号 学生姓名 科目名称 成绩)
-- 按成绩降序排序
SELECT s.studentno,studentname,subjectname,StudentResult
FROM student s
INNER JOIN result r
ON r.studentno = s.studentno
INNER JOIN `subject` sub
ON r.subjectno = sub.subjectno
WHERE subjectname='数据库结构-1'
ORDER BY StudentResult DESC
```

#### 分页

```mysql
/*============== 分页 ================
语法 : SELECT * FROM table LIMIT [offset,] rows | rows OFFSET offset
好处 : (用户体验,网络传输,查询压力)

推导:
   第一页 : limit 0,5
   第二页 : limit 5,5
   第三页 : limit 10,5
   ......
   第N页 : limit (pageNo-1)*pageSzie,pageSzie
   [pageNo:页码,pageSize:单页面显示条数]
   
*/

-- 每页显示5条数据
SELECT s.studentno,studentname,subjectname,StudentResult
FROM student s
INNER JOIN result r
ON r.studentno = s.studentno
INNER JOIN `subject` sub
ON r.subjectno = sub.subjectno
WHERE subjectname='数据库结构-1'
ORDER BY StudentResult DESC , studentno
LIMIT 0,5
```

### 集合查询

#### 嵌套查询

IN

IN后面返回的是表中满足约束条件的一个集合

![image-20210704175844153](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704175844153.png)

![image-20210704175858981](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704175858981.png)

其中：

![image-20210704180046542](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704180046542.png)

![image-20210704180053676](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704180053676.png)

#### 带ANY &ALL 的子查询

<img src="https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704181942161.png" alt="image-20210704181942161" style="zoom:150%;" />

ANY：![image-20210704182213219](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704182213219.png)

![image-20210704182218488](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704182218488.png)

ALL：

![image-20210704182240931](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704182240931.png)

![image-20210704182249462](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704182249462.png)

#### 带EXISTs 的子查询

not exists 如果后面的子查询没有值，返回1，否则为0

exists 如果后面的子查询有值，返回1，否则为0

#### 集合查询

用**UNION**并集 

![image-20210704183316920](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704183316920.png)

用**INTERSECT**交集

![image-20210704183336743](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704183336743.png)

用**EXCEPT**差集

<img src="https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704183421105.png" alt="image-20210704183421105" style="zoom: 200%;" />

### 数据的增、删、改

#### 数据插入

```mysql
INSERT INTO<表名>[(字段列表)] values(值列表)
```

```mysql
-- 指定所有字段：
INSERT INTO sb(id,dead,gender,email) VALUES(4,'2431','f','1243515');
-- 指定部分字段：
INSERT into sb(id,dead) VALUES(5,'242341');
-- 不指定字段：
INSERT INTO sb VALUES(6,'24453231','m','1515');
-- 批量添加：
INSERT INTO sb 
VALUES(6,'24453231','m','14355'),
(7,'3543231','m','1515'),
(8,'243451','m','5435'),
(9,'2543545331','m','13455');
```

#### 数据修改

```mysql
UPDATE<表名> SET <字段1>=<值1>,<字段2>=<值2>...
WHERE <约束条件>;
-- 将id 为 2 的 age改成34 ，sex改为'm'
UPDATE stu SET age=34,sex='m' WHERE id=2;
```

#### 数据删除

```mysql
DELETE FROM<表名>WHERE<条件>
-- 注意：筛选条件如不指定则删除该表的所有列数据
-- 删除stu表中id=10 的数据
DELETE FROM stu WHERE id=10;
```

```mysql
-- TRUNCATE命令
-- 用于完全清空表数据, 但表结构, 索引,约束等不变 ;
TRUNCATE [TABLE] table_name;
-- 清空年级表
TRUNCATE grade
```

**注意：TRUNCATE区别于DELETE命令**

- 相同 : 都能删除数据 , 不删除表结构 , 但TRUNCATE速度更快

- 不同 :

- - 使用TRUNCATE TABLE 重新设置AUTO_INCREMENT计数器
  - 使用TRUNCATE TABLE不会对事务有影响 （事务后面会说）

```mysql
-- 结论:truncate删除数据,自增当前值会恢复到初始值重新开始;不会记录日志.
-- 同样使用DELETE清空不同引擎的数据库表数据.重启数据库服务后
-- InnoDB : 自增列从初始值重新开始 (因为是存储在内存中,断电即失)
-- MyISAM : 自增列依然从上一个自增数据基础上开始 (存在文件中,不会丢失)
```

### 视图的概念、创建、修改、删除、查询

#### 视图概念

![image-20210704213337558](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704213337558.png)

#### 视图创建

```mysql
CREATE VIEW<视图名>[(<列名>[,<列名>]...)]
AS<子查询>
[WITH CHECK OPTION];
-- 简历信息系学生的视图并要求进行修改和插入操作时仍需保证该视图只有信息系的学生：
CREATE VIEW IS_STU
AS
SELECT * FROM Stu 
WHERE Stu.Sdept='IS'
WITH CHECK OPTION;
-- 后序对该视图进行增删改的时候，关系数据库管理系统会自动加上Stu.Sdept='IS'的条件
```

WITH CHECK OPTION 表示对视图进行UPDATE、INSERT和DELETE操作时要保证更新、插入或删除的满足视图定义中的谓词条件（即子查询中的条件表达式）

 ![image-20210704214823255](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704214823255.png) ![image-20210704214842183](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210704214842183.png)  

#### 视图删除

```mysql
DROP VIEW<视图名>[CASCADE]
```

视图删除后视图的定义将从数据字典中删除。如果该视图上还到处了其他视图，用CASCADE级联删除语句把由它导出的所有视图一并删除

#### 视图查询

```mysql
SELECT <字段名> FROM <视图名>;
-- 和表查询一样
```

#### 视图更新

因为视图是虚表，所以我们的对视图的操作最终要反映到对基本表的操作

```mysql
-- Ⅰ视图修改
UPDATE<视图名> SET <字段1>=<值1>,<字段2>=<值2>...
WHERE <约束条件>;
-- Ⅱ表修改
UPDATE<表名> SET <字段1>=<值1>,<字段2>=<值2>...
WHERE <约束条件>;-- 此处要再添加视图定义中子查询的约束条件
```

例：

```mysql
-- 将信息系学生视图IS_STU中学号为66的学生姓名改成'HR'
-- UPDATE VIEW
UPDATE IS_STU 
SET Sname='HR'
WHERE Sid=66;
-- UPDATE TABLE
UPDATE Stu
SET Sname='HR'
WHERE Sid=66 AND Sdept='IS';
```

## MySQL函数

### 常用函数

**数据函数**

```mysql
 SELECT ABS(-8);  /*绝对值*/
 SELECT CEILING(9.4); /*向上取整*/
 SELECT FLOOR(9.4);   /*向下取整*/
 SELECT RAND();  /*随机数,返回一个0-1之间的随机数*/
 SELECT SIGN(0); /*符号函数: 负数返回-1,正数返回1,0返回0*/
```

**字符串函数**

```mysql
 SELECT CHAR_LENGTH('狂神说坚持就能成功'); /*返回字符串包含的字符数*/
 SELECT CONCAT('我','爱','程序');  /*合并字符串,参数可以有多个*/
 SELECT INSERT('我爱编程helloworld',1,2,'超级热爱');  /*替换字符串,从某个位置开始替换某个长度*/
 SELECT LOWER('KuangShen'); /*小写*/
 SELECT UPPER('KuangShen'); /*大写*/
 SELECT LEFT('hello,world',5);   /*从左边截取*/
 SELECT RIGHT('hello,world',5);  /*从右边截取*/
 SELECT REPLACE('狂神说坚持就能成功','坚持','努力');  /*替换字符串*/
 SELECT SUBSTR('狂神说坚持就能成功',4,6); /*截取字符串,开始和长度*/
 SELECT REVERSE('狂神说坚持就能成功'); /*反转
 
 -- 查询姓周的同学,改成邹
 SELECT REPLACE(studentname,'周','邹') AS 新名字
 FROM student WHERE studentname LIKE '周%';
```

**日期和时间函数**

```mysql
 SELECT CURRENT_DATE();   /*获取当前日期*/
 SELECT CURDATE();   /*获取当前日期*/
 SELECT NOW();   /*获取当前日期和时间*/
 SELECT LOCALTIME();   /*获取当前日期和时间*/
 SELECT SYSDATE();   /*获取当前日期和时间*/
 
 -- 获取年月日,时分秒
 SELECT YEAR(NOW());
 SELECT MONTH(NOW());
 SELECT DAY(NOW());
 SELECT HOUR(NOW());
 SELECT MINUTE(NOW());
 SELECT SECOND(NOW());
```

**系统信息函数**

```mysql
 SELECT VERSION();  /*版本*/
 SELECT USER();     /*用户*/
```



### 聚合函数

| 函数名称 | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| COUNT()  | 返回满足Select条件的记录总和数，如 select count(*) 【不建议使用 *，效率低】 |
| SUM()    | 返回数字字段或表达式列作统计，返回一列的总和。               |
| AVG()    | 通常为数值字段或表达列作统计，返回一列的平均值               |
| MAX()    | 可以为数值字段，字符字段或表达式列作统计，返回最大的值。     |
| MIN()    | 可以为数值字段，字符字段或表达式列作统计，返回最小的值。     |

```mysql
 -- 聚合函数
 /*COUNT:非空的*/
 SELECT COUNT(studentname) FROM student;
 SELECT COUNT(*) FROM student;
 SELECT COUNT(1) FROM student;  /*推荐*/
 
 -- 从含义上讲，count(1) 与 count(*) 都表示对全部数据行的查询。
 -- count(字段) 会统计该字段在表中出现的次数，忽略字段为null 的情况。即不统计字段为null 的记录。
 -- count(*) 包括了所有的列，相当于行数，在统计结果的时候，包含字段为null 的记录；
 -- count(1) 用1代表代码行，在统计结果的时候，包含字段为null 的记录 。
 /*
 很多人认为count(1)执行的效率会比count(*)高，原因是count(*)会存在全表扫描，而count(1)可以针对一个字段进行查询。其实不然，count(1)和count(*)都会对全表进行扫描，统计所有记录的条数，包括那些为null的记录，因此，它们的效率可以说是相差无几。而count(字段)则与前两者不同，它会统计该字段不为null的记录条数。
 
 下面它们之间的一些对比：
 
 1）在表没有主键时，count(1)比count(*)快
 2）有主键时，主键作为计算条件，count(主键)效率最高；
 3）若表格只有一个字段，则count(*)效率较高。
 */
 
 SELECT SUM(StudentResult) AS 总和 FROM result;
 SELECT AVG(StudentResult) AS 平均分 FROM result;
 SELECT MAX(StudentResult) AS 最高分 FROM result;
 SELECT MIN(StudentResult) AS 最低分 FROM result;
```

**题目：**

```mysql
 -- 查询不同课程的平均分,最高分,最低分 ,平均分>80分
 -- 前提:根据不同的课程进行分组
 
 SELECT subjectname,AVG(studentresult) AS 平均分,MAX(StudentResult) AS 最高分,MIN(StudentResult) AS 最低分
 FROM result AS r
 INNER JOIN `subject` AS s
 ON r.subjectno = s.subjectno
 GROUP BY r.subjectno
 HAVING 平均分>80;
 
 /*
 聚合函数不能在where的条件当作条件 而要在having后使用
 where写在group by前面.
 要是放在分组后面的筛选
 要使用HAVING..
 因为having是从前面筛选的字段再筛选，而where是从数据表中的>字段直接进行的筛选的
 */
```

> MD5 加密

**一、MD5简介**

MD5即Message-Digest Algorithm 5（信息-摘要算法5），用于确保信息传输完整一致。是计算机广泛使用的杂凑算法之一（又译摘要算法、哈希算法），主流编程语言普遍已有MD5实现。将数据（如汉字）运算为另一固定长度值，是杂凑算法的基础原理，MD5的前身有MD2、MD3和MD4。

**二、实现数据加密**

新建一个表 testmd5

```mysql
 CREATE TABLE `testmd5` (
  `id` INT(4) NOT NULL,
  `name` VARCHAR(20) NOT NULL,
  `pwd` VARCHAR(50) NOT NULL,
  PRIMARY KEY (`id`)
 ) ENGINE=INNODB DEFAULT CHARSET=utf8
```

插入一些数据

```mysql
 INSERT INTO testmd5 VALUES(1,'kuangshen','123456'),(2,'qinjiang','456789')
```

如果我们要对pwd这一列数据进行**加密**，语法是：

```mysql
 update testmd5 set pwd = md5(pwd);
```

如果单独对某个用户(如kuangshen)的密码加密：

```mysql
 INSERT INTO testmd5 VALUES(3,'kuangshen2','123456')
 update testmd5 set pwd = md5(pwd) where name = 'kuangshen2';
```

插入新的数据自动加密

```mysql
 INSERT INTO testmd5 VALUES(4,'kuangshen3',md5('123456'));
```

查询登录用户信息（md5对比使用，查看用户输入加密后的密码进行比对）

```mysql
 SELECT * FROM testmd5 WHERE `name`='kuangshen' AND pwd=MD5('123456');
```



### 小结

```mysql
 -- ================ 内置函数 ================
 -- 数值函数
 abs(x)            -- 绝对值 abs(-10.9) = 10
 format(x, d)    -- 格式化千分位数值 format(1234567.456, 2) = 1,234,567.46
 ceil(x)            -- 向上取整 ceil(10.1) = 11
 floor(x)        -- 向下取整 floor (10.1) = 10
 round(x)        -- 四舍五入去整
 mod(m, n)        -- m%n m mod n 求余 10%3=1
 pi()            -- 获得圆周率
 pow(m, n)        -- m^n
 sqrt(x)            -- 算术平方根
 rand()            -- 随机数
 truncate(x, d)    -- 截取d位小数
 
 -- 时间日期函数
 now(), current_timestamp();     -- 当前日期时间
 current_date();                    -- 当前日期
 current_time();                    -- 当前时间
 date('yyyy-mm-dd hh:ii:ss');    -- 获取日期部分
 time('yyyy-mm-dd hh:ii:ss');    -- 获取时间部分
 date_format('yyyy-mm-dd hh:ii:ss', '%d %y %a %d %m %b %j');    -- 格式化时间
 unix_timestamp();                -- 获得unix时间戳
 from_unixtime();                -- 从时间戳获得时间
 
 -- 字符串函数
 length(string)            -- string长度，字节
 char_length(string)        -- string的字符个数
 substring(str, position [,length])        -- 从str的position开始,取length个字符
 replace(str ,search_str ,replace_str)    -- 在str中用replace_str替换search_str
 instr(string ,substring)    -- 返回substring首次在string中出现的位置
 concat(string [,...])    -- 连接字串
 charset(str)            -- 返回字串字符集
 lcase(string)            -- 转换成小写
 left(string, length)    -- 从string2中的左边起取length个字符
 load_file(file_name)    -- 从文件读取内容
 locate(substring, string [,start_position])    -- 同instr,但可指定开始位置
 lpad(string, length, pad)    -- 重复用pad加在string开头,直到字串长度为length
 ltrim(string)            -- 去除前端空格
 repeat(string, count)    -- 重复count次
 rpad(string, length, pad)    --在str后用pad补充,直到长度为length
 rtrim(string)            -- 去除后端空格
 strcmp(string1 ,string2)    -- 逐字符比较两字串大小
 
 -- 聚合函数
 count()
 sum();
 max();
 min();
 avg();
 group_concat()
 
 -- 其他常用函数
 md5();
 default();
```

## 数据库安全性

### 安全性概述

![image-20210705090618322](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705090618322.png)

> 基本命令

```mysql
/* 用户和权限管理 */ ------------------
用户信息表：mysql.user

-- 刷新权限
FLUSH PRIVILEGES

-- 增加用户 CREATE USER A IDENTIFIED BY '123456'
CREATE USER 用户名 IDENTIFIED BY [PASSWORD] 密码(字符串)
  - 必须拥有mysql数据库的全局CREATE USER权限，或拥有INSERT权限。
  - 只能创建用户，不能赋予权限。
  - 用户名，注意引号：如 'user_name'@'192.168.1.1'
  - 密码也需引号，纯数字密码也要加引号
  - 要在纯文本中指定密码，需忽略PASSWORD关键词。要把密码指定为由PASSWORD()函数返回的混编值，需包含关键字PASSWORD

-- 重命名用户 RENAME USER A TO A2
RENAME USER old_user TO new_user

-- 设置密码
SET PASSWORD = PASSWORD('密码')    -- 为当前用户设置密码
SET PASSWORD FOR 用户名 = PASSWORD('密码')    -- 为指定用户设置密码

-- 删除用户 DROP USER kuangshen2
DROP USER 用户名

-- 分配权限/添加用户
GRANT 权限列表 ON 库名.表名 TO 用户名 [IDENTIFIED BY [PASSWORD] 'password']
  - all privileges 表示所有权限
  - *.* 表示所有库的所有表
  - 库名.表名 表示某库下面的某表

-- 查看权限   SHOW GRANTS FOR root@localhost;
SHOW GRANTS FOR 用户名
   -- 查看当前用户权限
  SHOW GRANTS; 或 SHOW GRANTS FOR CURRENT_USER; 或 SHOW GRANTS FOR CURRENT_USER();

-- 撤消权限
REVOKE 权限列表 ON 库名.表名 FROM 用户名
REVOKE ALL PRIVILEGES, GRANT OPTION FROM 用户名    -- 撤销所有权限
```

> 权限解释

```mysql
-- 权限列表
ALL [PRIVILEGES]    -- 设置除GRANT OPTION之外的所有简单权限
ALTER    -- 允许使用ALTER TABLE
ALTER ROUTINE    -- 更改或取消已存储的子程序
CREATE    -- 允许使用CREATE TABLE
CREATE ROUTINE    -- 创建已存储的子程序
CREATE TEMPORARY TABLES        -- 允许使用CREATE TEMPORARY TABLE
CREATE USER        -- 允许使用CREATE USER, DROP USER, RENAME USER和REVOKE ALL PRIVILEGES。
CREATE VIEW        -- 允许使用CREATE VIEW
DELETE    -- 允许使用DELETE
DROP    -- 允许使用DROP TABLE
EXECUTE        -- 允许用户运行已存储的子程序
FILE    -- 允许使用SELECT...INTO OUTFILE和LOAD DATA INFILE
INDEX     -- 允许使用CREATE INDEX和DROP INDEX
INSERT    -- 允许使用INSERT
LOCK TABLES        -- 允许对您拥有SELECT权限的表使用LOCK TABLES
PROCESS     -- 允许使用SHOW FULL PROCESSLIST
REFERENCES    -- 未被实施
RELOAD    -- 允许使用FLUSH
REPLICATION CLIENT    -- 允许用户询问从属服务器或主服务器的地址
REPLICATION SLAVE    -- 用于复制型从属服务器（从主服务器中读取二进制日志事件）
SELECT    -- 允许使用SELECT
SHOW DATABASES    -- 显示所有数据库
SHOW VIEW    -- 允许使用SHOW CREATE VIEW
SHUTDOWN    -- 允许使用mysqladmin shutdown
SUPER    -- 允许使用CHANGE MASTER, KILL, PURGE MASTER LOGS和SET GLOBAL语句，mysqladmin debug命令；允许您连接（一次），即使已达到max_connections。
UPDATE    -- 允许使用UPDATE
USAGE    -- “无权限”的同义词
GRANT OPTION    -- 允许授予权限
```

### 安全性控制

![image-20210705090712449](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705090712449.png)

![image-20210705091013060](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705091013060.png)

REFERENCE权限代表是否允许创建外键

#### 数据库授权 GRANT

![image-20210705091304843](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705091304843.png)

```mysql
-- 让用户user1拥有对表Stu的列id的查询权限
GRANT SELECT ON Stu(id) TO user1 WITH GRANT OPTION;
```

#### 数据库回收权限 REVOKE

![image-20210705091330370](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705091330370.png)

加了CASCADE就会把user1授权给其他用户的权限也一并回收

```mysql
-- 回收用户user1对表Stu的列id的查询权限
REVOKE SLELECT ON Stu(id) FROM user1;
```

#### 数据库角色

角色指的是一类人，可以给一类人授权

![image-20210705092914461](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705092914461.png)

####   视图授权

![image-20210705093204314](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705093204314.png)

### 审计

![image-20210705093751140](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705093751140.png)

### 数据加密

把明文变成密文

## MySQL备份



## 数据库完整性

### 三大完整性

![image-20210705103319868](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705103319868.png)

![image-20210705110129264](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705110129264.png)

![image-20210705110554656](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705110554656.png)

### 断言

![image-20210705110715604](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705110715604.png)

![image-20210705110810048](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705110810048.png)

![image-20210705110731872](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705110731872.png)

### 触发器

![image-20210705111508762](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705111508762.png)

![image-20210705111522964](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705111522964.png)

![image-20210705111714268](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705111714268.png)

![image-20210705112612355](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705112612355.png)

这里书上有误： 书上插入StudentInsertLog的是每次对Student插入操作后学生的总数

```mysql
REFERENCING
	OLD TABLE AS O
	NEW TABLE AS N
FOR EACH STATEMENGT
	INSERT INTO StudentInsertLog(Numbers) -- 这里写VALUES?
	SELECT(
	(SELECT COUNT(*) FROM N)-(SELECT COUNT(*) FROM O)
    );
```

## 关系数据库理论

### 引入范式的原因

1. 数据冗余
2. 插入异常
3. 更新异常
4. 删除异常

![image-20210705135025995](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705135025995.png)

### 依赖

![image-20210705135039176](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705135039176.png)

![image-20210705135053089](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705135053089.png)

### 候选码

概念：能推出所有属性的码

![image-20210705135407009](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705135407009.png)

![image-20210705135419352](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705135419352.png)

![image-20210705135503865](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705135503865.png)

### 三大范式

**第一范式 (1st NF)**

第一范式的目标是确保每列的原子性,如果每列都是不可再分的最小数据单元,则满足第一范式

**第二范式(2nd NF)**

第二范式（2NF）是在第一范式（1NF）的基础上建立起来的，即满足第二范式（2NF）必须先满足第一范式（1NF）。

第二范式要求每个表只描述一件事情

**第三范式(3rd NF)**

如果一个关系满足第二范式,并且除了主键以外的其他列都不传递依赖于主键列,则满足第三范式.

第三范式需要确保数据表中的每一列数据都和主键直接相关，而不能间接相关。

#### 1NF

![image-20210705135912848](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705135912848.png)

![image-20210705135931908](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705135931908.png)

#### 2NF

![image-20210705140017750](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705140017750.png)

![image-20210705140050212](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705140050212.png)

<img src="https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705140307658.png" alt="image-20210705140307658" style="zoom:50%;" />

<img src="https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705140313838.png" alt="image-20210705140313838" style="zoom:50%;" />

<img src="https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705140411108.png" alt="image-20210705140411108" style="zoom:50%;" />

#### 3NF

![image-20210705140518592](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705140518592.png)

![image-20210705140528731](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705140528731.png)

#### BNCF

![image-20210705141402098](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705141402098.png)

![image-20210705141426707](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705141426707.png)

![image-20210705141534198](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705141534198.png)

### 最小依赖集

![image-20210705141916429](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705141916429.png)

![img](https://pic2.zhimg.com/80/v2-c0dc4cddaecee8bfce2d2ec09a11f77e_720w.jpg?source=1940ef5c)

<img src="https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705144055776.png" alt="image-20210705144055776" style="zoom: 80%;" />



例题

![image-20210705142021813](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705142021813.png)

![image-20210705144001890](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705144001890.png)

### 模式分解

![image-20210705143453633](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705143453633.png)

求保持函数依赖的3NF分解  

![image-20210705143544064](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705143544064.png)

![image-20210705143552754](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705143552754.png)

![image-20210705145144350](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705145144350.png)

## 数据库设计

**步骤**

![image-20210705145606687](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705145606687.png)

### ER图

![image-20210705145715202](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705145715202.png)

![image-20210705145724796](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705145724796.png)

![image-20210705151726900](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705151726900.png)

![image-20210705151746009](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705151746009.png)

![image-20210705151806803](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705151806803.png)

例：

![image-20210705152853877](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705152853877.png)

注：1.ER图 2.创建的相应表（M对N关系单独建立一个表）

## 数据库恢复技术

### 事务![image-20210705160343696](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705160343696.png)

> **基本语法**

```mysql
-- 使用set语句来改变自动提交模式
SET autocommit = 0;   /*关闭*/
SET autocommit = 1;   /*开启*/

-- 注意:
--- 1.MySQL中默认是自动提交
--- 2.使用事务时应先关闭自动提交

-- 开始一个事务,标记事务的起始点
START TRANSACTION  

-- 提交一个事务给数据库
COMMIT

-- 将事务回滚,数据回到本次事务的初始状态
ROLLBACK

-- 还原MySQL数据库的自动提交
SET autocommit =1;

-- 保存点
SAVEPOINT 保存点名称 -- 设置一个事务保存点
ROLLBACK TO SAVEPOINT 保存点名称 -- 回滚到保存点
RELEASE SAVEPOINT 保存点名称 -- 删除保存点
```

```mysql
CREATE DATABASE `shop`CHARACTER SET utf8 COLLATE utf8_general_ci;
USE `shop`;

CREATE TABLE `account` (
`id` INT(11) NOT NULL AUTO_INCREMENT,
`name` VARCHAR(32) NOT NULL,
`cash` DECIMAL(9,2) NOT NULL,
PRIMARY KEY (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8

INSERT INTO account (`name`,`cash`)
VALUES('A',2000.00),('B',10000.00)

-- 转账实现
SET autocommit = 0; -- 关闭自动提交
START TRANSACTION;  -- 开始一个事务,标记事务的起始点
UPDATE account SET cash=cash-500 WHERE `name`='A';
UPDATE account SET cash=cash+500 WHERE `name`='B';
COMMIT; -- 提交事务
# rollback;
SET autocommit = 1; -- 恢复自动提交
```



### 故障种类

![image-20210705161531110](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705161531110.png)

### 恢复方式

![image-20210705161702413](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705161702413.png)

### 恢复策略

![image-20210705162048762](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705162048762.png)

![image-20210705162244545](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705162244545.png)

## 并发控制

![image-20210705162636405](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705162636405.png)

![image-20210705162940399](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705162940399.png)

![image-20210705164907193](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705164907193.png)

上图依次是 一级 三级 二级锁。

### 可串行性

![image-20210705164946902](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210705164946902.png)

## MYSQL索引优化

+ hash： 内存上存储，只能等值查询
+ 二叉树：树身过深造成io次数多影响数据读取效率
+ b树：内节点放data，相对b+树能存放的节点少，变相增加了树深度
+ b+树：每个内节点包含更多的节点，降低树的高度，镜数据范围变成多个区间，去建越多检索越快；叶子节点两两指针互相连接符合此派的预读特性，顺序查找的性能更高

一般来说，B+Tree比B-Tree更适合实现外存储索引结构。在B+Tree的每个叶子节点增加一个指向相邻叶子节点的指针，就形成了带有顺序访问指针的B+Tree。做这个优化的目的是为了提高区间访问的性能，

b+树上有两个头指针，一个指向根节点，另一个指向关键字最小的叶子节点（数据节点），且所有叶子节点之间是一种链式环结构。因此可以对b+树进行两种查找运算：

+ 对于主键的范围查找和分页查找
+ 从根节点开始的随即查找

### 存取原理

索引一般以文件形式存储在磁盘上，索引检索需要磁盘I/O操作。与主存不同，磁盘I/O存在机械运动耗费，因此磁盘I/O的时间消耗是巨大的。

当需要从磁盘读取数据时，磁头需要移动对准相应磁道，这个过程叫做寻道，所耗费时间叫做寻道时间，然后磁盘旋转将目标扇区旋转到磁头下，这个过程耗费的时间叫做旋转时间。

为了达到这个目的，磁盘往往不是严格按需读取，而是每次都会预读，即使只需要一个字节，磁盘也会从这个位置开始，顺序向后读取一定长度的数据放入内存。这样做的理论依据是计算机科学中著名的局部性原理：

+ 当一个数据被用到时，其附近的数据也通常会马上被使用。

+ 程序运行期间所需要的数据通常比较集中。

由于磁盘顺序读取的效率很高（不需要寻道时间，只需很少的旋转时间），因此对于具有局部性的程序来说，预读可以提高I/O效率。

预读的长度一般为页（page）的整倍数。硬件及操作系统往往将主存和磁盘存储区分割为连续的大小相等的块，每个存储块称为一页，主存和磁盘以页为单位交换数据。当程序要读取的数据不在主存中时，会触发一个缺页异常，此时系统会向磁盘发出读盘信号，磁盘会找到数据的起始位置并向后连续读取一页或几页载入内存中，然后异常返回，程序继续运行。

### B-/+Tree索引的性能分析

先从B-Tree分析，根据B-Tree的定义，可知检索一次最多需要访问h个节点。数据库系统的设计者巧妙利用了磁盘预读原理，将一个节点的大小设为等于一个页，这样每个节点只需要一次I/O就可以完全载入。为了达到这个目的，在实际实现B-Tree还需要使用如下技巧：

每次新建节点时，直接申请一个页的空间，这样就保证一个节点物理上也存储在一个页里，加之计算机存储分配都是按页对齐的，就实现了一个node只需一次I/O。

B-Tree中一次检索最多需要h-1次I/O（根节点常驻内存）。出度d是非常大的数字，通常超过100，因此h非常小（通常不超过3）。综上所述，用B-Tree作为索引结构效率是非常高的。

B+Tree更适合外存索引，原因和内节点出度d有关。从上面分析可以看到，d越大索引的性能越好，而出度的上限取决于节点内key和data的大小。由于B+Tree内节点去掉了data域，因此可以拥有更大的出度，拥有更好的性能。

### MySQL索引实现

在MySQL中，索引属于存储引擎级别的概念，不同存储引擎对索引的实现方式是不同的，本文主要讨论MyISAM和InnoDB两个存储引擎的索引实现方式。

#### MyISAM索引实现

MyISAM引擎使用B+Tree作为索引结构，叶节点的data域存放的是数据记录的地址。下图是MyISAM索引的原理图：

![image-20210711153225922](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210711153225922.png)

这里设表一共有三列，假设我们以Col1为主键，则图8是一个MyISAM表的主索引（Primary key）示意。可以看出MyISAM的索引文件仅仅保存数据记录的地址。在MyISAM中，主索引和辅助索引（Secondary key）在结构上没有任何区别，只是主索引要求key是唯一的，而辅助索引的key可以重复。如果我们在Col2上建立一个辅助索引，则此索引的结构如下图所示：

![image-20210711153300307](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210711153300307.png)

MyISAM中索引检索的算法为首先按照B+Tree搜索算法搜索索引，如果指定的Key存在，则取出其data域的值，然后以data域的值为地址，读取相应数据记录。

MyISAM的索引方式也叫做“非聚集”的，之所以这么称呼是为了与InnoDB的聚集索引区分。

#### InnoDB索引实现

虽然InnoDB也使用B+Tree作为索引结构，但具体实现方式却与MyISAM截然不同。

第一个重大区别是InnoDB的数据文件本身就是索引文件。从上文知道，MyISAM索引文件和数据文件是分离的，索引文件仅保存数据记录的地址。而在InnoDB中，表数据文件本身就是按B+Tree组织的一个索引结构，这棵树的叶节点data域保存了完整的数据记录。这个索引的key是数据表的主键，因此InnoDB表数据文件本身就是主索引。

![image-20210711153507632](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210711153507632.png)

图10是InnoDB主索引（同时也是数据文件）的示意图，可以看到叶节点包含了完整的数据记录。这种索引叫做聚集索引。因为InnoDB的数据文件本身要按主键聚集，所以InnoDB要求表必须有主键（MyISAM可以没有），如果没有显式指定，则MySQL系统会自动选择一个可以唯一标识数据记录的列作为主键，如果不存在这种列，则MySQL自动为InnoDB表生成一个隐含字段作为主键。

第二个与MyISAM索引的不同是InnoDB的辅助索引data域存储相应记录主键的值而不是地址。换句话说，InnoDB的所有辅助索引都引用主键作为data域。例如，图为定义在Col3上的一个辅助索引：

![image-20210711153607994](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210711153607994.png)

聚集索引这种实现方式使得按主键的搜索十分高效，但是辅助索引搜索需要检索两遍索引：首先检索辅助索引获得主键，然后用主键到主索引中检索获得记录。

知道了InnoDB的索引实现后，就很容易明白为什么不建议使用过长的字段作为主键，因为所有辅助索引都引用主索引，过长的主索引会令辅助索引变得过大。再例如，用非单调的字段作为主键在InnoDB中不是个好主意，因为InnoDB数据文件本身是一颗B+Tree，非单调的主键会造成在插入新记录时数据文件为了维持B+Tree的特性而频繁的分裂调整（会有io），十分低效，而使用自增字段作为主键则是一个很好的选择。

|                | MYISAM     | InnoDB                     |
| -------------- | ---------- | -------------------------- |
| 索引类型       | 非聚簇     | 聚簇                       |
| 支持事务       | N          | Y                          |
| 支持表锁       | Y          | Y                          |
| 支持行锁       | N          | Y                          |
| 支持外键       | N          | Y                          |
| 支持全文索引   | N          | Y（5.6后）                 |
| 适合的操作类型 | 大量select | 大量insert、delete、update |

### 索引相关基础知识

**业务主键与自然主键**

与业务相挂钩的叫自然主键，与业务无关的叫代理主键。代理主键最好选择自增：唯一，且在维护索引时尽量减少触发页分裂和页合并。

如果表使用自增主键，那么每次插入新的记录，记录就会顺序添加到当前索引节点的后续位置，当一页写满，就会自动开辟一个新的页。这样就会形成一个紧凑的索引结构，近似顺序填满。由于每次插入时也不需要移动已有数据，因此效率很高，也不会增加很多开销在维护索引上。

如果使用非自增主键（如果身份证号或学号等），由于每次插入主键的值近似于随机，因此每次新纪录都要被插到现有索引页得中间某个位置。

#### **回表**

1. 先通过辅助（普通）索引定位到主键值id；

2. 在通过主（聚集）索引定位到行记录；

这就是所谓的**回表查询**，先定位主键值，再定位行记录，它的性能较扫一遍索引树更低。

![image-20210711162734181](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210711162734181.png)

#### **覆盖索引**

MySQL官网，类似的说法出现在explain查询计划优化章节，即explain的输出结果Extra字段为Using index时，能够触发索引覆盖：只需要在一棵索引树上就能获取SQL所需的所有列数据，无需回表，速度更快。

实现索引覆盖常见的方法是：将被查询的字段，建立到联合索引里去。

例子：

*create table user (*

*id int primary key,*

*name varchar(20),*

*sex varchar(5),*

*index(name)*

*)engine=innodb;*

第一个SQL语句：

*select id,name from user where name='shenjian';*

能够命中name索引，索引叶子节点存储了主键id，通过name的索引树即可获取id和name，无需回表，符合索引覆盖，效率较高。

第二个SQL语句：

*select id,name**,sex** from user where name='shenjian';*

能够命中name索引，索引叶子节点存储了主键id，但sex字段必须回表查询才能获取到，不符合索引覆盖，需要再次通过id值扫码聚集索引获取sex字段，效率会降低。

如果把(name)单列索引升级为联合索引(name, sex)就不同了。

*create table user (*

*id int primary key,*

*name varchar(20),*

*sex varchar(5),*

*index(name, sex)*

*)engine=innodb;*

那么：

*select id,name ... where name='shenjian';*

*select id,name,**sex** ... where name='shenjian';*

都能够命中索引覆盖，无需回表。

#### 最左匹配

索引的底层是一颗B+树，那么联合（组合）索引当然还是一颗B+树，只不过联合索引的健值数量不是一个，而是多个。构建一颗B+树只能根据一个值来构建，因此数据库依据联合索引最左的字段来构建B+树。

![image-20210711165238035](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210711165238035.png)

在a值相等的情况下，b值又是按顺序排列的，但是这种顺序是相对的。所以最左匹配原则遇上范围查询就会停止，剩下的字段都无法使用索引。例如a = 1 and b = 2 a,b字段都可以使用索引，因为在a值确定的情况下b是相对有序的，而a>1and b=2，a字段可以匹配上索引，但b值不可以，因为a的值是一个范围，在这个范围中b是无序的。

#### 索引下推

不使用索引条件下推优化时存储引擎通过索引检索到数据，然后返回给MySQL服务器，服务器然后判断数据是否符合条件。
当使用索引条件下推优化时，如果存在某些被索引的列的判断条件时，MySQL服务器将这一部分判断条件传递给存储引擎，然后由存储引擎通过判断索引是否符合MySQL服务器传递的条件，只有当索引符合条件时才会将数据检索出来返回给MySQL服务器。索引条件下推优化可以减少存储引擎查询基础表的次数，也可以减少MySQL服务器从存储引擎接收数据的次数。

#### 索引匹配方式

```mysql
alter table staffs add index idx_nap(name, age, pos);
```

**全值匹配**：全值匹配指的是和索引中的全部列进行匹配

```mysql
explain select * from staffs where name = ‘July’ and age = ‘23’ and pos = ‘dev’;
```

**匹配最左前缀**：只匹配前面的几列

```mysql
explain select * from staffs where name = ‘July’ and age = ‘23’;
```

**匹配列前缀**：能够匹配某一列的值的开头部分

```mysql
explain select * from staffs where name like ‘J%’;
```

**匹配范围值**：能够查找某一个范围的数据

```mysql
explain select * from staffs where name > ‘Mary’;
```

**精确匹配某一列并范围匹配另一列**：能够查询第一列的所有和第二列的部分

```mysql
explain select * from staffs where name = ‘July’ and age > 25;
```

**只访问索引的查询**：查询的时候只须要访问索引，不须要访问数据行，本质上就是覆盖索引

### 通过索引进行优化

#### 哈希索引

基于哈希表的实现，只有精确匹配索引所有列的等值查询才有效。数据文件保存在内存中，不适合范围查找。用在memory存储引擎中显示支持。

#### 聚集索引和非聚集索引

不是单独的索引类型，而是指：

聚簇索引：数据行和相邻的键值紧凑存储在一起(.frm&.idb)

非聚簇索引：数据文件和索引文件分开存放（.frm&.MYD&.MYI）

#### 覆盖索引

上面说过

#### 优化细节⭐

+ 索引查询的时候尽量不要用表达式，把计算放在业务层面而不是数据库层面（表达式不能使用索引，它要先进行运算而不是先去B+树查找，B+树查找过程是没有运算过程的，MYSQL不会做优化）
+ 尽量使用主键查询，而不是其他索引，因为主键查询不触发回表查询（IO问题）
+ 使用前缀索引（有时索引时很长的字符串，这会让索引变得大且慢，通常可以使用某个列开始的部分字符这样能大大节约索引空间，不过会降低索引的选择性）
+ 使用索引扫描来排序（MYSQL有两种方式可以生成有序的结果：通过排序操作或按照索引顺序扫描，如果explain出来的type列为index，则说明mysql使用了索引扫描来做排序）（扫描索引本身是很快的，因为只需要从一条索引记录移动到下一条，但是如果索引不能覆盖查询所需的所有列，就不得不扫描每一条索引记录就回表查询一次，这都是IO，因此按照顺序读取数据的速度通常要比顺序全表扫描慢）（只有当索引的列顺序和order by子句的顺序完全一致且所有列的排序方式都一样，MYSQL才能使用索引来对结果进行培训；如果查询关联多张表，则只有当orderby子句引用的字段全部为第一张表才能使用索引排序。order by子句和查找型插叙的限制是一样的，需要满嘴索引的最左前缀要求，否则无法利用索引排序）
+ union all、or、in都能使用索引，推荐in，in最快
+ 范围列能使用到索引（各种>=<和between）,但范围列后面的列不能使用索引，索引最多用于一个范围列
+ 强制类型转换会全表扫描（比如某字段phoneNum是varchar类型，phoneNum=1234 触发强制类型转换 ，phoneNum=‘1234’不触发强制类型转换）
+ 更新频繁且区分度不是特别高的字段不宜建立索引，维护时会频繁触发io操作
+ 建立索引的列不允许为null
+ 当进行表连接的时候最好别超过3张，因为需要join字段。数据类型必须一致
+ 能用limit尽量用limit，减少io量
+ 单表索引建议控制在5个内，否则io量大
+ 单索引字段不允许超过5个（建立组合索引时，别加太多字段）
+ 做优化前必须了解清楚表结构

## MYSQL性能分析工具（运维层面）

*show profile剖析单条语句功能*

**查看是否支持**：

```mysql
show VARIABLES LIKE '%profil%'
```

have_profiling          YES    表示支持profile
profiling                  OFF     表示没开启
profiling_history_size  15     就最近15条



**开启profiling=1**命令行执行：

```mysql
set profiling =1
```

**执行一个SQL语句**

```mysql
select * from user5  -- (随便执行一个就行)
```

**查看query_id:**

```mysql
show profiles ;-- 看到每个sql语句执行的时间
show profile for query query_id; -- 查看该sql语句执行的过程中线程的状态和在每个状态消耗的时间

```

 
