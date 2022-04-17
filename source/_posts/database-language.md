---
title: 「数据库」数据库语言SQL
date: 2018-08-27 19:36:31
tags: [数据库, 后端开发]
categories: 数据库
---

### SQL语言概述
**结构化查询语言**(Structured Query Language)简称SQL，是一种特殊目的的编程语言，是一种数据库查询和程序设计语言，用于存取数据以及查询、更新和管理关系数据库系统。
+ SQL语言是集DDL、DML和DCL于一体的数据库语言<!-- more -->
    1. **DDL语句**引导词：Create(建立)，Alter(修改)，Drop(撤消)
        - 模式的定义和删除，包括定义Database，Table，View，Index,完整性约束条件等，也包括定义对象(RowType行对象，Type列对象)
    2. **DML语句**引导词：Insert ，Delete, Update, Select
        - 各种方式的更新与检索操作，如直接输入记录，从其他Table(由SubQuery建立)输入
        - 各种复杂条件的检索，如连接查找，模糊查找，分组查找，嵌套查找等
        - 各种聚集操作，求平均、求和、…等，分组聚集，分组过滤等
    3. **DCL语句**引导词：Grant，Revoke
        - 安全性控制：授权和撤消授权


### 1. 利用SQL建立数据库
DDL：数据定义语言（Data Definition Language)，
DDL通常由**DBA(数据库管理员)**来使用，也有经DBA授权后由应用程序员来使用
1. 创建数据库(DB)：**Create Database**
    + 数据库(Database)是若干具有相互关联关系的Table/Relation的集合
    + 简单语法形式：`create database database 数据库名;`
2. 创建DB中的Table(定义关系模式)：**Create Table**
    + `Create table 表名(列名 数据类型 [Primary key|Unique] [Not null][,列名 数据类型 [Not null], …]);`
        - `[]`表示其括起的内容可以省略，`|`表示其隔开的两项可取其一
        - `Primary key`: 主键约束。每个表只能创建一个主键约束
        - `Unique`: 唯一性约束(即候选键)。可以有多个唯一性约束
        - `Not null`: 非空约束。
3. **数据类型**（SQL-92标准）
    + `char(n)`:固定长度的字符串
    + `varchar(n)`:可变长字符串
    + `int`:整数 //有时不同系统也写作integer
    + `numeric(p，q)`:固定精度数字，小数点左边p位，右边(p-q)位
    + `real`:浮点精度数字 //有时不同系统也写作`float(n)`，小数点后保留n位
    + `date`:日期 (如 2003-09-12)
    + `time`:时间 (如 23:15:003)
> 注意: 现行商用DBMS的数据类型有时有些差异


### 2. 利用SQL简单查询
DML：数据操纵语言（Data Manipulation Language)，
DML通常由**用户或应用程序员**使用，访问经授权的数据库
1. 向Table中添加数据(追加元组)：**Insert into**
    + **`insert into insert into 表名[(列名[, 列名] …] values (值[,值], …);`**
        - values值的排列，须与列名排列一致
        - 若所有列名省略，则values值的排列须与该表存储中的列名排列一致

2. 单表查询**Select**
    + **`Select Select 列名[[,列名] …] From 表名[Where 检索条件];`**
        - 语义：从表名所给出的表中，查询出满足检索条件的元组，并按给定的列名及顺序进行投影显示。
        - 相当于：`Π[列名,...,列名](σ检索条件(表名))`
    + Select语句中的select … , from… , where…, 等被称为子句，在以上基本形式基础上会增加许多构成要素，也会增加许多新的子句，满足不同的需求。

3. 检索条件的书写**Where**
    + 与选择运算`σF(R)`的条件F书写一样，只是其逻辑运算符用 and,or,not 来表示, 同时也要注意运算符的优先次序及括弧的使用。书写要点是注意对自然语言检索条件的正确理解。
    + `Select Tname From Teacher Where Salary > 2000 and D# = ’03’;`//检索教师表中所有工资大于2000元 并且是03系的教师姓名

4. 排重(`DISTINCT`)
    + 关系模型不允许出现重复元组。但现实DBMS，却允许出现重复元组。
    + 在Table中要求无重复元组是通过定义Primary key或Unique来保证的;
    + 而在检索结果中要求无重复元组, 是通过**DISTINCT保留字**的使用来实现的。
    + `Select DISTINCT S# From SC Where Score > 80; `

5. 排序(`ORDER BY`)
    + Select语句中结果排序是通过增加**order by**子句实现的
    + `order by 列名 [asc|desc]`
    + 意义为检索结果按指定列名进行排序，若后跟asc或省略，则为升序；若后跟desc, 则为降序。

6. 模糊查询(`*LIKE*`)
    + `_`：一个字符，`%`：任意长度字符。
    + `Select Sname From Student Where Sname Like '张_ _';`//检索名字为张某某的所有同学姓名
    + `Select Sname From Student Where Sname Not Like '张%';`//检索名字不姓张的所有同学姓名


### 3. 利用SQL多表联合查询
多表联合检索可以通过连接运算来完成，而连接运算又可以通过广义笛卡尔积后再进行选择运算来实现。
+ 检索语句: **`Select 列名[[,列名] …] From 表名1,表名2,… Where 检索条件;`**
+ 相当于`Π[列名,...,列名](σ检索条件(表名1 × 表名2 × …))`
+ 检索条件中要包含连接条件，通过不同的连接条件可以实现等值连接、不等值连接及各种θ-连接

1. θ-连接之**等值连接**
    + 多表连接时，如两个表的属性名相同，则需采用**`表名.属性名`**方式来限定该属性是属于哪一个表
    + `Select Sname From Student, SC Where Student.S#=SC.S# and SC.C#='001' Order By Score DESC;`//按“001”号课成绩由高到低顺序显示所有学生的姓名(二表连接)

2. 属性重名重名处理(表别名)
    + 连接运算涉及到重名的问题，如两个表中的属性重名，连接的两个表重名(同一表的连接)等，因此需要使用**`别名`**以便区分
    + `Select 列名 as 列别名[[,列名 as 列别名] …] From 表名1 as 表别名1,表名2 as 表别名2,… Where Where 检索条件;`
    + 当定义了别名后，在检索条件中可以使用别名来限定属性
    + as 可以省略

3. θ-连接之**不等值连接**
    + `Select T1.Tname as Teacher1, T2.Tname as Teacher2 From Teacher T1, Teacher T2 Where T1.Salary>T2.Salary;`//求有薪水差额的任意两位教师

4. 实例：
    + `Select S1.S# From SC S1, SC S2 Where S1.S# = S2.S# and S1.C#='001' and S2.C#='002' and S1.Score > S2.Score;`//求“001”号课成绩比“002”号课成绩高的所有学生的学号


### 4. 利用SQL进行增-删-改
1. SQL-之**更新操作**
    + 元组新增Insert：新增一个或一些元组到数据库的Table中
    + 元组更新Update: 对某些元组中的某些属性值进行重新设定
    + 元组删除Delete：删除某些元组

>- SQL-DML既能单一记录操作，也能对记录集合进行批更新操作
>- SQL-DML之更新操作需要利用前面介绍的子查询(Subquery)的概念，以便处理“一些”、“某些”等

2. SQL-之**INSERT**
    + 单一元组新增命令形式：插入一条指定元组值的元组
        - **`insert into 表名 [(列名[,列名]…)] values (值 [,值]…);`**
    + 批数据新增命令形式：插入子查询结果中的若干条元组。待插入的元组由子查询给出。
        - **`insert into 表名 [(列名[，列名]…)] 子查询;`**
        - 示例：`Insert Into St (S#,Sname) Select S#,Sname From Student Where Sname like '%伟';`//将检索到的满足条件的同学新增到该表中

> 注意：当新增元组时，DBMS会检查用户定义的完整性约束条件等，如不符合完整性约束条件，则将不会执行新增动作。

3. SQL-之**DELETE**
    + 元组删除Delete命令: 删除满足指定条件的元组
    + **`Delete From 表名 [ Where 条件表达式];`**
    + 如果Where条件省略，则删除所有的元组(清空表)。
    + 示例：`Delete From Student Where S# in ( Select S# From SC Where Score < 60 Group by S# Having Count(*)>= 4);`//删除有四门不及格课程的所有同学

4. SQL-之**UPDATE**
    + 元组更新Update命令: 用指定要求的值更新指定表中满足指定条件的元组的指定列的值
    + **`Update 表名 Set 列名=表达式 | (子查询) [[,列名=表达式 | (子查询) ] …] [ Where 条件表达式];`**
    + 如果Where条件省略，则更新所有的元组。
    + 示例：`Update Teacher Set Salary=Salary*1.1 Where D# in (Select D# From Dept Where Dname='计算机');`//将所有计算机系的教师工资上调10%


### 5. 利用SQL语言修正与撤销数据库
1. 修正基本表的定义
    + **`alter table tablename`**
    + **`[add {colname datatype, …}]`** //增加新列
    + **`[drop {完整性约束名}]`** //删除完整性约束
    + **`[modify {colname datatype, …}]`** //修改列定义
    + 示例：`Alter Table Student Drop Unique(Sname);`删除学生姓名必须取唯一值的约束
    + 示例：`Alter Table Student Add Saddr char[40],PID char[18];`在学生表Student上增加二列Saddr, PID

2. SQL-DDL之撤销与修改
    + `drop table 表名;` //撤消基本表
    + `drop database 数据库名;` //撤消数据库

3. SQL-DDL之数据库指定与关闭命令
    + 有些DBMS提供了操作多个数据库的能力，此时在进行数据库操作时需要指定待操作数据库与关闭数据库的功能。
    + `use 数据库名;` //指定当前数据库
    + `close 数据库名;` //关闭当前数据库


### 6. SQL Server介绍
SQL Server 是 Microsoft提供的一款关系数据库管理系统
1. SQL Server 的系统数据库
    + Master：是SQL Server中最重要的系统数据库，存储SQL Server中的元数据。
    + Model：模板数据库，在创建新的数据库时，SQL Server将会复制此数据库作为新数据库的基础。
    + Msdb：代理服务数据库，提供一个存储空间。
    + Tempdb：临时数据库，为所有的临时表、临时存储过程及其他临时操作提供存储空间，断开连接时，临时表与存储过程自动被删除。
2. SQL Server的数据库
    + 文件：有三种文件扩展名：.mdf、.ndf、.ldf
        - 主数据库文件：扩展名为.mdf，是存储数据库的启动信息和部分或全部数据。一个数据库可以有多个数据库文件，但主数据库文件只有一个。
        - 辅助数据文件：扩展名为.ndf，用于放置主数据库文件中所定义数据库的其它数据，可有多个。在数据庞大时，可以帮助存储数据。
        - 日志文件：扩展名.ldf。每个数据库至少有一个事务日志文件。
    + 页面：是SQL Server存储的最小单位。一页为8K或8192字节。
    + 空间(extent)：是8个连续的页面，即64K数据，是分配数据表存储空间的一种单位

#### 6.1 SQL Server数据库的创建-删除与维护
1. 创建数据库
    + 语法形式：Create Database 库名
    + 可视化操作(查询分析器)：Database(鼠标右键) -> new Database… -> 填写数据库名及配置
    + 创建数据库的过程就是为数据库设计名称、设计所占用存储空间和存 放文件位置的过程。特别是在网络数据库中，对数据库的设计显得尤为重要。如估计数据可能占用的磁盘空间有多大，日志文件及其他要占用多大空间。
    + 创建数据库的用户自动成为数据库的拥有者。
2. 删除数据库
    + 语法形式：Drop Database 库名
    + 可视化操作(查询分析器)：数据库名(鼠标右键) -> Delete
    + 对不再需要的数据库，应删除以释放空间。删除的结果将是所有数据库文件都一并被删除。
    + 当数据库处于正在使用或正在恢复状态时，不能删除。
3. 备份数据库
    + 可视化操作(查询分析器)：数据库名(鼠标右键) -> Tasks -> Back Up…
    + 备份就是对数据库或事务日志进行备份。SQL的备份是动态的，备份的过程还可以让用户继续改写。只有系统管理员、数据库的拥有者及数据库的备份者才有权限进行数据备份。可以通过企业管理器进行数据库备份。
        - 完全数据库备份：完全备份数据文件和日志文件。
        - 差异备份（增量备份）：对最近一次数据库备份以来发生的数据变化进行备份。这要在完全备份的基础上进行。特点是速度快。
        - 事务日志备份：对数据库发生的事务进行备份。包括从上次进行事务日志备份、差异备份和数据库完全备份之后，所有已经完成的事务。能尽可能的恢复最新的数据库记录。特点是所需磁盘空间小，时间少。
        - 数据库文件和文件组备份：用在数据库相当大的情况下。
4. 恢复数据库
    + 可视化操作(查询分析器)：数据库名(鼠标右键) -> Tasks -> Restore
    + 数据库的恢复是指将数据库备份加载到系统中的过程。在根据数据库备份文件恢复过程中，系统将自动执行安全性检查、重建数据库结构及完成填写数据库内容。
    + 数据库的恢复是静态的。所以在恢复前，应将需要恢复的数据库访问属性设为单用户，不要让其他用户操作。
    + 可以通过企业管理器来完成数据库恢复。
5. 数据库授权: 
    + 语法形式：grant 权限 on 表名 to 用户名
    + 权限有：select,update,insert,delete,exec,dri。
    + 对被授权的用户，要先成为该数据库的使用者，即要把用户加到数据库里,才能授权.

#### 6.2 SQL Server数据表的创建-与增/删/改/查
1. 创建表
    + 同一用户不能建立同一个表名的表，同一表名的表可有多个拥有者。但在使用时，需要在这些表上加上所有者的表名。
    + 用T-SQL语句创建表，语法形式：`CREATE TABLE [数据库名.所有者名.]表名 ({<列名 数据类型>} [缺省值][约束][是否为空] …)`
        > 注意：T-SQL是SQL Server软件的SQL语言，与标准版有些差异。但标准版SQL，一般情况下SQL Server软件也都支持
    + 可视化操作(查询分析器)：数据库名 -> Tables -> New Table…
2. 增加、修改表字段
    + 语法形式：`ALTER TABLE ADD | ALTER 字段名 <类型>`
3. 创建、删除与修改约束
    + 约束是SQL提供自动保持数据库完整性的一种方法，共5种。
    + 用T-SQL语句建立约束，语法形式：`CONSTRAINT 约束名 约束类型 (列名)`
        - 约束名：在库中应该唯一，如不指定，系统会给出
        - 约束类型 (5种)：
            * primary key constraint (主键值)
            * unique constraint (唯一性)
            * check constraint (检查性)
            * default constraint (默认)
            * foreign key constraint (外部键)
        - 列名：要约束的字段名
    + 示例:`Create Table Course ( C# char(3) , Cname char(12), Chours integer, Credit float(1), T# char(3) ) constraint pk primary key(C# ));`


### 7. SQL语言-子查询
- 子查询：出现在Where子句中的Select语句被称为子查询(subquery) , 子查询返回了一个集合，可以通过与这个集合的比较来确定另一个查询集合。
- 三种类型的子查询：(NOT) IN-子查询；θ-Some/θ-All子查询；(NOT) EXISTS子查询

#### 7.1 (NOT) IN子查询
1. 基本语法：`表达式 [not] in (子查询)`
    + 语法中，表达式的最简单形式就是列名或常数。
    + 语义：判断某一表达式的值是否在子查询的结果中。
    + 示例：
        - `Select * From Student Where Sname in ('张三', '王三');`//列出张三、王三同学的所有信息
        - `Select S#, Sname From Student Where S# in (Select S# From SC Where C#='001');`//列出选修了001号课程的学生的学号和姓名
3. 非相关子查询：内层查询独立进行，没有涉及任何外层查询相关信息的子查询前面的子查询示例都是非相关子查询
4. 相关子查询：内层查询需要依靠外层查询的某些参量作为限定条件才能进行的子查询
5. 外层向内层传递的参量需要使用外层的表名或表别名来限定
    + 示例：`Select Sname From Student Stud Where S# in ( Select S# From SC Where S# = Stud.S# and C#='001');`//求学过001号课程的同学的姓名

> 注意：相关子查询只能由外层向内层传递参数，而不能反之；这也称为变量的作用域原则。


#### 7.2 θ-Some/θ-All子查询
1. 基本语法：`表达式 θ some (子查询)` / `表达式 θ all (子查询)`
    + 语法中，θ是比较运算符：`<, >, >=, <=, =, <>`。
    + 语义：将表达式的值与子查询的结果进行比较：
        - 如果表达式的值至少与子查询结果的某一个值相比较满足 关系，则`表达式 θ some (子查询)`的结果便为真
        - 如果表达式的值与子查询结果的所有值相比较都满足 关系，则`表达式 θ all (子查询)`的结果便为真
    + 示例：
        - `Select Tname From Teacher Where Salary <= all ( Select Salary From Teacher);`//找出工资最低的教师姓名
        - `Select S# From SC Where C# = “001” and Score < some ( Select Score From SC Where C#='001');`//找出001号课成绩不是最高的所有学生的学号

> 在SQL标准中，也有θ-Any 谓词，但由于其语义的模糊性：any, “任一”是指所有呢？还是指某一个？不清楚，所以被θ-Some替代以求更明晰。

2. 等价性变换需要注意
    + `表达式 = some (子查询)`和`表达式 in (子查询)`含义**相同**
    + `表达式 <> some (子查询)`和`表达式 not in (子查询)`含义**不同**
    + `表达式 <> all (子查询)`和`表达式 not in (子查询)`含义**相同**


#### 7.3 (NOT) EXISTS子查询
1. 基本语法：`[not] Exists [not] Exists (子查询)`
    + 语义：子查询结果中有无元组存在

``` sql
--示例：检索选修了赵三老师主讲课程的所有同学的姓名
Select DISTINCT Sname From Student
    Where exists ( Select * From SC, Course, Teacher
        Where SC.C#=Course.C# and SC. S#=Student.S#
        and Course.T# = Teacher.T# and Tname='赵三');

--示例：检索学过001号教师主讲的所有课程的所有同学的姓名
Select Sname From Student
    Where not exists //不存在
        ( Select * From Course //有一门001教师主讲课程
        Where Course.T# = ‘001’ and not exists //该同学没学过
            ( Select * From SC
            Where S# = Student.S# and C# = Course.C#));
--上述语句的意思：不存在有一门001号教师主讲的课程该同学没学过
```


### 8. SQL语言-结果计算与聚集计算
#### 8.1 结果计算
Select-From-Where语句中，Select子句后面不仅可是列名，而且可是一些计算表达式或聚集函数，表明在投影的同时直接进行一些运算
+ `Select Select 列名 | expr | agfunc(列名) [[, 列名 | expr | agfunc(列名) ] … ] From 表名1 [, 表名2 … ] [ Where Where 检索条件 ];`
    - expr可以是常量、列名、或由常量、列名、特殊函数及算术运算符构成的算术运算式。特殊函数的使用需结合各自DBMS的说明书
    - agfunc()是一些聚集函数

``` sql
--示例：求有差额(差额>0)的任意两位教师的薪水差额
Select T1.Tname as TR1, T2.Tname as TR2, T1.Salary – T2.Salary
    From Teacher T1, Teacher T2
    Where T1.Salary > T2.Salary;
```

#### 8.2 聚集函数
SQL提供了五个作用在简单列值集合上的内置聚集函数agfunc, 分别是：COUNT、SUM、AVG、MAX、MIN

|聚合函数 |支持的数据类型|描述|
|--------|------------|---|
|count() |任何类型/*   |计算结果集中的总行数|
|sum()   |Numeric     |计算指定列中所有非空值的总和|
|avg()   |numeric     |计算指定列中所有非空值的平均值|
|max()   |char/numeric|返回指定列中最大值|
|min()   |char/numeric|返回指定列中最小值|

``` sql
--示例：求教师的工资总额
Select Sum(Salary) From Teacher;
--示例：求计算机系教师的工资总额
Select Sum(Salary) From Teacher T, Dept
    Where Dept.Dname = ‘计算机’ and Dept.D# = T.D#;
--示例：求数据库课程的平均成绩
Select AVG(Score) From Course C, SC
    Where C.Cname = ‘数据库’ and C.C# = SC.C#;
```


### 9. SQL语言-分组查询与分组过滤
#### 9.1 分组查询
分组：SQL可以将检索到的元组按照某一条件进行分类，具有相同条件值的元组划到一个组或一个集合中，同时处理多个组或集合的聚集运算。
1. 分组的基本语法：
``` sql
Select Select 列名 | expr | agfunc(列名) [[, 列名 | expr | agfunc(列名) ] … ]
    From 表名1 [, 表名2 … ]
    [ Where Where 检索条件 ]
    [ Group by Group by 分组条件 ] ;
```

2. 分组条件可以是：`列名1, 列名2, …`
3. 示例： 求每一个学生的平均成绩
    + `Select S#, AVG(Score) From SC Group by S#;`

#### 9.2 分组过滤
聚集函数是不允许用于Where子句中的：Where子句是对每一元组进行条件过滤，而不是对集合进行条件过滤
* 分组过滤：若要对集合(即分组)进行条件过滤，即满足条件的集合/分组留下，不满足条件的集合/分组剔除。
* Having子句，又称分组过滤子句。需要有Group by子句支持，换句话说，没有Group by子句，便不能有Having子句。

1. 基本语法：
``` sql
Select Select 列名 | expr | agfunc(列名) [[, 列名 | expr | agfunc(列名) ] … ]
    From 表名1 [, 表名2 … ]
    [ Where Where 检索条件 ]
    [ Group by Group by 分组条件 [ Having Having 分组过滤条件] ] ;
```

2. 示例：求不及格课程超过两门的同学的学号
    + `Select S# From SC Where Score<60 Group by S# Having Count(*)>2;` 

#### 9.3 where子句与having子句的区别
1. 聚合函数是比较where、having 的关键。在from后面的执行顺序：
    + `where -> 聚合函数(sum,min,max,avg,count) ->having`
2. 列出group by来比较二者:
    + where子句：是在分组之前使用，表示从所有数据中筛选出部分数据，以完成分组的要求，在where子句中不允许使用统计函数，没有group by子句也可以使用。
    + having子句：是在分组之后使用的，表示对分组统计后的数据执行再次过滤，可以使用统计函数，有group by子句之后才可以出现having子句。

> 注意事项 ： 
> 1. where 后不能跟聚合函数，因为where执行顺序大于聚合函数。 
> 2. where 子句的作用是在对查询结果进行分组前，将不符合where条件的行去掉，即在分组之前过滤数据，条件中不能包含聚组函数，使用where条件显示特定的行。 
> 3. having 子句的作用是筛选满足条件的组，即在分组之后过滤数据，条件中经常包含聚组函数，使用having 条件显示特定的组，也可以使用多个分组标准进行分组。



### 10. SQL语言实现关系代数操作
SQL语言：并运算UNION, 交运算INTERSECT, 差运算EXCEPT。
+ 基本语法形式：
    - `子查询 {Union [ALL] | Intersect [ALL] | Except [ALL] 子查询}`
+ 通常情况下自动删除重复元组：不带ALL。若要保留重复的元组，则要带ALL。
    * 假设子查询1的一个元组出现m次，子查询2的一个元组出现n次，则该元组在：
        - 子查询1 Union ALL 子查询2 ，出现m + n次
        - 子查询1 Intersect ALL 子查询2 ，出现min(m,n)次
        - 子查询1 Except ALL 子查询2 ，出现max(0, m – n)次

> UNION运算符是Entry-SQL92的一部分, INTERSECT、EXCEPT运算符是Full-SQL92的一部分,它们都是Core-SQL99的一部分，但**有些DBMS并不支持**这些运算，使用时要注意。


#### 10.1 SQL并运算(UNION)
1. 示例：已知两个表
    * Customers(Cid, Cname, City, Discnt)
    * Agents(Aid, Aname, City, Percent)
2. 求客户所在的或者代理商所在的城市
``` sql
Select City From Customers
UNION
Select City From Agents;
```


#### 10.2 SQL交运算(INTERSECT)
1. 示例：求既学过002号课，又学过003号课的同学学号
``` sql
Select S# From SC Where C# = ‘002’
INTERSECT
Select S# From SC Where C# = ‘003’;
```
2. 上述语句也可采用如下不用INTERSECT的方式来进行
    + `Select S# From SC Where C# = ‘002’ and S# IN (Select S# From SC Where C# = ‘003’);`

3. 交运算符Intersect并没有增强SQL的表达能力，没有Intersect， SQL也可以用其他方式表达同样的查询需求。只是有了Intersect更容易表达一些，但增加了SQL语言的不唯一性。


#### 10.3 SQL差运算(EXCEPT)
1. 示例： 假定所有学生都有选课，求没学过002号课程的学生学号
``` sql
Select DISTINCT S# From SC
EXCEPT
Select S# From SC Where C# = ‘002’;
```

2. 上述语句也可采用如下不用INTERSECT的方式来进行
``` sql
Select DISTINCT S# From SC SC1
    Where not exists ( Select * From SC
        Where C# = ‘002’ and S# = SC1.S#);
```

3. 差运算符Except也没有增强SQL的表达能力，没有Except， SQL也可以用其他方式表达同样的查询需求。只是有了Except更容易表达一些，但增加了SQL语言的不唯一性。


#### 10.4 空值的处理
空值是其值不知道、不确定、不存在的值；数据库中有了空值，会影响许多方面，如影响聚集函数运算的正确性，不能参与算术、比较或逻辑运算等
1. 在SQL标准中和许多现流行的DBMS中，空值被用一种特殊的符号Null来标记，使用特殊的空值检测函数来获得某列的值是否为空值。
2. 空值检测：
    + `is [not ] null` //测试指定列的值是否为空值
3. 示例：找出年龄值为空的学生姓名
    + `Select Sname From Student Where Sage is null;`
4. 现行DBMS的空值处理小结
    + 除is [not] null之外，空值不满足任何查找条件
    + 如果null参与算术运算，则该算术表达式的值为null
    + 如果null参与比较运算，则结果可视为false。在SQL-92中可看成unknown
    + 如果null参与聚集运算，则除count(*)之外其它聚集函数都忽略null


#### 10.5 内连接、外连接
1. 标准SQL语言中连接运算通常为：
    + `Select Select 列名[[,列名]… ] From 表名1,表名2,… Where 检索条件;`
    + 即相当于采用`Π[列名,…,列名](σ 检索条件(表名1 × 表名2 × …))`。
2. SQL的高级语法中引入了内连接与外连接运算，具体形式：
``` sql
Select Select 列名 [ [, 列名] … ]
    From 表名1 [NATURAL]
    [ INNER | { LEFT | RIGHT | FULL} [OUTER]] JOIN 表名2
    { ON 连接条件 | Using (Colname {, Colname …}) }
    [ Where Where 检索条件 ] … ;
```
3. 由 **连接类型** 和 **连接条件** 构成连接运算。
    + **`Natural`**：出现在结果关系中的两个连接关系的元组在公共属性上取值相等，且公共属性只出现一次
    + **`Inner Join`**: 即关系代数中的θ-连接运算
    + **`Left Outer Join, Right Outer Join, Full Outer Join`**: 即关系代数中的外连接运算
    + **`on <连接条件>`**：出现在结果关系中的两个连接关系的元组取值满足连接条件，且公共属性出现两次
    + **`using (Col1, Col2, …, Coln)`**：Col是两个连接关系的公共属性的子集，元组在(Col1,Col2,…,Coln)上取值相等，且(Col1,Col2,…,Coln)只出现一次
4. 示例:

``` sql
-- (Inner Join)求所有教师的任课情况并按教师号排序(没有任课的教师也需列在表中)
Select Teacher.T#, Tname, Cname
    From Teacher Inner Join Course
        ON Teacher.T# = Course.T#
    Order by Teacher.T# ASC;

--(Outer Join)求所有教师的任课情况(没有任课的教师也需列在表中)
Select Teacher. T#, Tname, Cname
    From Teacher Left Outer Join Course
        ON Teacher.T# = Course.T#
    Order by Teacher.T# ASC ;
```


### 11. SQL语言之视图及其应用
1. 数据库的三级模式两层映像
    * 三级模式：数据库系统是由外模式、模式(概念模式)和内模式三级构成
    * 应用--> **外模式**(多个) --> **概念模式**(一个) --> **内模式**(一个) --> 数据库
    * 两层映像：`E-C`映像(外模式->概念模式)、`C-I`映像(概念模式->内模式)。
2. 对应概念模式的数据在SQL中被称为**基本表(Table)**, 而对应外模式的数据称为**视图(View)**。**视图不仅包含外模式，而且包含其E-C映像**。
3. **基本表**是实际存储于存储文件中的表，基本表中的**数据是需要存储的**
4. **视图**在SQL中只存储其由基本表导出视图所需要的公式，即由基本表产生视图的映像信息，其**数据并不存储**，而是在运行过程中动态产生与维护的
5. 对视图数据的更改最终要反映在对基本表的更改上。

#### 11.1 视图的定义
视图需要“先定义，再使用”；定义视图，有时可方便用户进行检索操作。
1. 定义视图: `create view view_name [(列名[列名] …)] as 子查询 [with check option]`
    + 如果视图的属性名缺省，则默认为子查询结果中的属性名；也可以显式指明其所拥有的列名。
    + with check option指明当对视图进行insert，update，delete时，要检查进行insert/update/delete的元组是否满足视图定义中子查询中定义的条件表达式
2. 示例：定义一个视图 CompStud 为计算机系的学生，通过该视图可以将Student表中其他系的学生屏蔽掉
``` sql
Create View CompStud AS
    (Select * From Student
        Where D# in (Select D# From Dept
            Where Dname = ‘计算机’));
```

#### 11.2 视图的使用
使用视图：定义好的视图，可以像Table一样，在SQL各种语句中使用
+ 示例：检索计算机系的所有学生，我们可使用CompStud
    - `Select * From CompStud;`
+ 示例：检索计算机系的年龄小于20的所有学生，我们可使用CompStud
    - `Select * From CompStud Where Sage<20;`

#### 11.3 视图的更新
SQL视图更新：是比较复杂的问题，因视图不保存数据，对视图的更新最终要反映到对基本表的更新上，而有时，视图定义的映射不是可逆的。
1. SQL视图更新的可执行性
    + 如果视图的select目标列包含聚集函数，则不能更新
    + 如果视图的select子句使用了unique或distinct，则不能更新
    + 如果视图中包括了group by子句，则不能更新
    + 如果视图中包括经算术表达式计算出来的列，则不能更新
    + 如果视图是由单个表的列构成，但并没有包括主键，则不能更新
2. 对于由单一Table子集构成的视图，即如果视图是从单个基本表使用选择、投影操作导出的，并且包含了基本表的主键，则可以更新
3. 可更新SQL视图示例：

``` sql
-- 定义视图
create view CStud(S#, Sname, Sclass)
as ( select S#, Sname, Sclass from Student where D# ='03');
-- 更新视图
Insert into CStud Values ('98030104', '张三丰', '980301');
-- 更新视图 将转换为 更新基本表
insert into Student values ('98030104', '张三丰', Null, Null, '03', '980301')
```

#### 11.4 视图的撤销
已经定义的视图也可以撤消
* 撤消视图：`Drop View view_name`

不仅视图可以撤消，基本表、数据库等都可以撤消
* 撤消基本表：`Drop Table 表名`



### 12. 数据库完整性
数据库完整性(DB Integrity)是指：DBMS应保证的DB的一种特性--在任何情况下的正确性、有效性和一致性
* 广义完整性：语义完整性、并发控制、安全控制、DB故障恢复等
* 狭义完整性：专指语义完整性，DBMS通常有专门的完整性管理机制与程序来处理语义完整性问题。

#### 12.1 基本概念
关系模型中有完整性要求：实体完整性、参照完整性、用户自定义完整性
1. 数据库完整性管理的作用
    * 防止和避免数据库中不合理数据的出现
    * DBMS应尽可能地自动防止DB中语义不合理现象
    * 如DBMS不能自动防止，则需要应用程序员和用户在进行数据库操作时处处加以小心，每写一条SQL语句都要考虑是否符合语义完整性，这种工作负担是非常沉重的，因此应尽可能多地让DBMS来承担
2. DBMS怎样自动保证完整性：
    * DBMS允许用户定义一些完整性约束规则(用SQL-DDL来定义)
    * 当有DB更新操作时，DBMS自动按照完整性约束条件进行检查，以确保更新操作符合语义完整性
3. **完整性约束条件**(或称完整性约束规则)的一般形式：Integrity Constraint::=(O,P,A,R)
    + O：数据集合：约束的对象(列、多列(元组)、元组集合)
    + P：谓词条件：需要定义什么样的约束
    + A：触发条件：默认更新时检查
    + R：响应动作：默认拒绝

#### 12.2 数据库完整性的分类
1. 按约束对象分类:
    - 域完整性约束条件：施加于某一列上，对给定列上所要更新的某一候选值是否可以接受进行约束条件判断，这是孤立进行的
    - 关系完整性约束条件：施加于关系/table上，对给定table上所要更新的某一候选元组是否可以接受进行约束条件判断，或是对一个关系中的若干元组和另一个关系中的若干元组间的联系是否可以接受进行约束条件判断

2. 按约束来源分类:
    - 结构约束：来自于模型的约束，例如函数依赖约束、主键约束(实体完整性)、外键约束(参照完整性)，只关心数值相等与否、是否允许空值等；
    - 内容约束：来自于用户的约束，如用户自定义完整性，关心元组或属性的取值范围。例如Student表的Sage属性值在15岁至40岁之间等。

3. 按约束状态分类:
    - 静态约束：要求DB在任一时候均应满足的约束；例如Sage在任何时候都应满足大于0而小于150(假定人活最大年龄是150)。
    - 动态约束：要求DB从一状态变为另一状态时应满足的约束；例如工资只能升，不能降：工资可以是800元，也可以是1000元；可以从800元更改为1000元，但不能从1000元更改为800元。


### 13. 数据库的静态完整性(约束)
1. SQL语言支持的约束类别：
    + 静态约束
        - 列完整性—域完整性约束
        - 表完整性--关系完整性约束
    + 动态约束
        - 触发器

2. Create Table有三种功能：定义关系模式、定义完整性约束 和定义物理存储特性
    + 定义完整性约束条件：列完整性、表完整性

3. 列约束：一种**域约束类型**，对单一列的值进行约束
``` sql
{ NOT NULL |                  //列值非空
[ CONSTRAINT constraintname ] //为约束命名，便于以后撤消
{ UNIQUE                      //列值是唯一
| PRIMARY KEY                 //列为主键
| CHECK (search_cond)         //列值满足条件,条件只能使用列当前值
| REFERENCES tablename [(colname) ]
[ON DELETE { CASCADE | SET NULL } ] } } 
```

4. 表约束：一种**关系约束类型**，对多列或元组的值进行约束
``` sql
[ CONSTRAINT constraintname ]       //为约束命名，便于以后撤消
{ UNIQUE (colname {,colname…})      //几列值组合在一起是唯一
| PRIMARY KEY (colname {,colname…}) //几列联合为主键
| CHECK (search_condition)          //元组多列值共同满足条件
                                    //条件中只能使用同一元组的不同列当前值
| FOREIGN KEY (colname {,colname…})
REFERENCES tablename [(colname {,colname…})]//引用另一表tablename的若干列的值作为外键
```
> check中的条件可以是Select-From-Where内任何Where后的语句，包含子查询。

5. Create Table中定义的表约束或列约束可以在以后根据需要进行撤消或追加。撤消或追加约束的语句是 Alter Table(不同系统可能有差异)
    + 示例：撤消SC表的ctscore约束(由此可见，未命名的约束是不能撤消)
        - `Alter Table SC DROP CONSTRAINT ctscore;`
    + 有些DBMS支持独立的追加约束, 注意书写格式可能有些差异
        - 示例：`Alter Table SC Add Constraint nctscore check (Score>=0.0 and Score<=150.0));`

6. 现约束的方法-断言ASSERTION
    + 一个断言就是一个谓词表达式，它表达了希望数据库总能满足的条件
    + 表约束和列约束就是一些特殊的断言
    + SQL还提供了复杂条件表达的断言。其语法形式为：
        - `CREATE ASSERTION <assertion-name> CHECK <predicate>`
    + 当一个断言创建后，系统将检测其有效性，并在每一次更新中测试更新是否违反该断言。

``` sql
-- 示例: “每个分行的贷款总量必须小于该分行所有账户的余额总和”
create assertion sum_constraint check
    (not exists (select * from branch
    where (select sum(amount ) from loan
        where loan.branch_name = branch.branch_name )
    >= (select sum (balance ) from account
        where account.branch_name = branch.branch_name )))
-- 数据表：
account(branch_name, account_number,…, balance) //分行，账户及其余额
loan(branch_name , loan_number, amount,) //分行的每一笔贷款
branch(branch_name, … ) //分行
```
> 断言测试增加了数据库维护的负担，要小心使用复杂的断言。


### 14. 数据库的动态完整性(触发器)
实现数据库动态完整的方法—触发器Trigger
1. 触发器Trigger
    + Create Table中的表约束和列约束基本上都是静态的约束，也基本上都是对单一列或单一元组的约束(尽管有参照完整性)，为实现动态约束以及多个元组之间的完整性约束，就需要触发器技术Trigger
    + Trigger是一种过程完整性约束(相比之下，Create Table中定义的都是非过程性约束),是一段程序，该程序可以在特定的时刻被自动触发执行，比如在一次更新操作之前执行，或在更新操作之后执行。

2. 基本语法
``` sql
CREATE TRIGGER trigger_name BEFORE | AFTER
    { INSERT | DELETE | UPDATE [OF colname {, colname...}] }
    ON tablename [REFERENCING corr_name_def {, corr_name_def...} ]
    [FOR EACH ROW | FOR EACH STATEMENT]
                //对更新操作的每一条结果(前者)，或整个更新操作完成(后者)
    [WHEN (search_condition)]           //检查条件，如满足执行下述程序
    { statement         //单行程序直接书写，多行程序要用下行方式
    | BEGIN ATOMIC statement; { statement;...} END }
```

3. 触发器Trigger意义：
    + 当某一事件发生时(Before|After),对该事件产生的结果(或是每一元组，或是整个操作的所有元组), 检查条件`search_condition`,如果满足条件，则执行后面的程序段。条件或程序段中引用的变量可用`corr_name_def`来限定。

4. 事件：BEFORE | AFTER { INSERT | DELETE | UPDATE …}
    + 当一个事件(Insert, Delete, 或Update)发生之前Before或发生之后After触发
    + 操作发生，执行触发器操作需处理两组值：更新前的值和更新后的值，这两个值由`corr_name_def`的使用来区分

5. `corr_name_def`的定义
``` sql
{ OLD [ROW] [AS] old_row_corr_name //更新前的旧元组命别名为
| NEW [ROW] [AS] new_row_corr_name //更新后的新元组命别名为
| OLD TABLE [AS] old_table_corr_name //更新前的旧Table命别名为
| NEW TABLE [AS] new_table_corr_name //更新后的新Table命别名为
}
```
> `corr_name_def`将在检测条件或后面的动作程序段中被引用处理

6. 示例1: 设计一个触发器当进行Teacher表更新元组时, 使其工资只能升不能降
``` sql
create trigger teacher_chgsal before update of salary
    on teacher
    referencing new x, old y
    for each row when (x.salary < y.salary)
begin
    raise_application_error(-20003, 'invalid salary on update');
    //此条语句为Oracle的错误处理函数
end;
```

7. 示例2: 假设student(S#, Sname, SumCourse), SumCourse为该同学已学习课程的门数，初始值为0，以后每选修一门都要对其增1 。设计一个触发器自动完成上述功能。
``` sql
create trigger sumc after insert on sc
    referencing new row newi
    for each row
begin
    update student set SumCourse = SumCourse + 1
    where S# = :newi.S# ;
end;
```

8. 示例3：假设student(S#, Sname, SumCourse), 当删除某一同学S#时，该同学的所有选课也都要删除。设计一个触发器完成上述功能
``` sql
create trigger delS# after delete on Student
    referencing old oldi
    for each row
begin
    delete sc where S# = :oldi.S# ;
end; 
```


### 15. 数据库索引
索引是对数据库表中一列或多列的值进行排序的一种**数据结构**（最常见的是B-Tree）
1. 索引的作用
    1. 快速取数据；
    2. 保证数据记录的唯一性；
    3. 实现表与表之间的参照完整性；
    4. 在使用ORDER by、group by子句进行数据检索时，利用索引可以减少排序和分组的时间。
2. 创建索引：`CREATE INDEX  索引名称  on 表名(字段名);`
3. 删除索引：`DROP INDEX 索引名称`
4. 索引注意事项：
    1. 查询时减少使用`*`返回全部列，不要返回不需要的列
    2. where表达式子句包含索引的表达式置前
    3. 避免在Order by中使用表达式
    4. 索引技术是数据库自动使用，一个表格只存在一个索引就够了
5. 缺点
    1. 索引的缺点是创建和维护索引需要耗费时间和空间
    2. 索引可以提高查询速度，会减慢写入速度
    3. 索引并不是越多越好，索引固然可以提高相应的 select 的效率，但同时也降低了 insert 及 update 的效率，因为 insert 或 update 时有可能会重建索引，所以怎样建索引需要慎重考虑，视具体情况而定。一个表的索引数最好不要超过6个，若太多则应考虑一些不常使用到的列上建的索引是否有必要。

#### 15.1 索引主要种类
根据数据库的功能，可以在数据库设计器中创建三种索引：唯一索引、主键索引和聚集索引。提示：尽管唯一索引有助于定位信息，但为获得最佳性能结果，建议改用主键或唯一约束。
1. 唯一索引
    + 唯一索引是不允许其中任何两行具有相同索引值的索引。当现有数据中存在重复的键值时，大多数数据库不允许将新创建的唯一索引与表一起保存。数据库还可能防止添加将在表中创建重复键值的新数据。例如，如果在employee表中职员的姓(lname)上创建了唯一索引，则任何两个员工都不能同姓。
2. 主键索引
    + 数据库表经常有一列或多列组合，其值唯一标识表中的每一行。该列称为表的主键。在数据库关系图中为表定义主键将自动创建主键索引，主键索引是唯一索引的特定类型。该索引要求主键中的每个值都唯一。当在查询中使用主键索引时，它还允许对数据的快速访问。
3. 聚集索引
    + 在聚集索引中，表中行的物理顺序与键值的逻辑（索引）顺序相同。一个表只能包含一个聚集索引。如果某索引不是聚集索引，则表中行的物理顺序与键值的逻辑顺序不匹配。与非聚集索引相比，聚集索引通常提供更快的数据访问速度。聚集索引和非聚集索引的区别，如字典默认按字母顺序排序，读者如知道某个字的读音可根据字母顺序快速定位。因此聚集索引和表的内容是在一起的。如读者需查询某个生僻字，则需按字典前面的索引，举例按偏旁进行定位，找到该字对应的页数，再打开对应页数找到该字。这种通过两个地方而查询到某个字的方式就如非聚集索引。
4. 索引列
    + 可以基于数据库表中的单列或多列创建索引。多列索引可以区分其中一列可能有相同值的行。如果经常同时搜索两列或多列或按两列或多列排序时，索引也很有帮助。例如，如果经常在同一查询中为姓和名两列设置判据，那么在这两列上创建多列索引将很有意义。


### 16. 数据库序列
序列(SEQUENCE)是序列号生成器，可以为表中的行自动生成序列号，产生一组等间隔的数值(类型为数字)。其主要的用途是生成表的主键值，可以在插入语句中引用，也可以通过查询检查当前值，或使序列增至下一个值。创建序列需要`CREATE SEQUENCE`系统权限。
#### 16.1 Oracle中的序列（Sequence）
1. 创建序列
``` sql
create sequence 序列名 
    [increment by n]   --每次增加n个，默认为1
    [start with n]     --起始值n，默认为1
    [{maxvalue n | nomaxvalue}]  --最大值设置，递增默认10的27次方，递减默认-1
    [{minvalue n | nominvalue}]  --最小值设置，递增默认1，递减默认-10的26次方
    [{cycle | nocycle}]   --是否循环
    [{cache n | nocache}] --是否对序列进行内存缓冲，默认为20
```

2. 查询序列
    + `NEXTVAL`:返回序列中下一个有效的值，任何用户都可以引用。
    + `CURRVAL`:中存放序列的当前值,NEXTVAL 应在 CURRVAL 之前指定 ，二者应同时有效。

``` sql
--查询下一个将要使用的序列
select 序列名.nextval from dual
--查询当前序列
select 序列名.currval from dual 
```
>- Oracle将sequence的定义存储在数据字典之中。
>- Sequence是独立于事务的，就是说序列的增加不需要等待事务的完成，也就是说序列是异步于事务而增长的。这说明，你访问不到别的用户使用该sequence产生的值，也就是说你只能访问到你当前产生的值，即使其他用户已经增加了sequence的值；还说明如果事务回滚，sequence不会回滚，它所发生的改变是一维的。

3. 删除序列：`Drop sequence 序列名`
4. 更改序列：`Alter sequence 序列名 [其余参数同创建序列]`
5. 使用序列示例：

``` sql
-- 1.直接使用
insert into person (id, name, password) values (序列名.nextval, '张三', '123')

-- 2.也可以通过建立触发器，当有数据插入表person时，使用oracle序列为其去的递增的主键值
-- 2.1创建触发器
create or replace trigger 触发器名 before insert on person
for each row
begin
    select 序列名.nextval into :new.id from dual;
end;
-- 2.2插入数据
insert into person ( username, age, password) values ('张三', 20, 'zhang123')
```

6. 注意点：
    + 一个序列可以被多张别使用，不过一般建议为每个表建立单独的序列。
    + 当使用到序列的事务发生回滚。会造成序列号不连续。在用生成的序列值作为编号做插入数据库操作时，可能遇到事务提交失败，从而导致序号不连续。
    + 大量语句发生请求，申请序列时，为了避免序列在运用层实现序列而引起的性能瓶颈。Oracle序列允许将序列提前生成 n个先存入内存，在发生大量申请序列语句时，可直接到运行最快的内存中去得到序列。但cache个数最好不要设置过大，因为在数据库重启时，会清空内存信息，预存在内存中的序列会丢失，当数据库再次启动后，序列从上次内存中最大的序列号+1 开始存入n个。这种情况也能会在数据库关闭时也会导致序号不连续。


#### 16.2 Mysql中的序列（AUTO_INCREMENT）
MySQL中最简单使用序列的方法就是使用`AUTO_INCREMENT`来定义列。
1. orale没有类似mysql的AUTO_INCREMENT这样的自增长字段，实现插入一条记录，自动增加1.oracle是通过sequence（序列）来完成的。
2. 首先mysql的自增长“序列”和序列是两回事，mysql本身不提供序列机制。
3. mysql的AUTO_INCREMENT可以设置起始值，但是不能设置步长，其步长默认就是1.
4. mysql一个表只能有一个自增长字段。自增长只能被分配给固定表的固定的某一字段，不能被多个表共用。并且只能是数字型。


### 17. 数据库安全性
数据库安全性是指DBMS应该保证的数据库的一种特性(机制或手段)：免受非法、非授权用户的使用、泄漏、更改或破坏
1. 数据库安全性管理涉及许多方面
    1. 社会法律及伦理方面：私人信息受到保护，未授权人员访问私人信息会违法
    2. 公共政策/制度方面：例如，政府或组织的信息公开或非公开制度
    3. 安全策略：政府、企业或组织所实施的安全性策略，如集中管理和分散管理，需者方知策略(也称最少特权策略)
    4. 数据的安全级别: 绝密(Top Secret), 机密(Secret),可信(Confidential)和无分类(Unclassified)
    5. 数据库系统DBS的安全级别：物理控制、网络控制、操作系统控制、DBMS控制
2. DBMS的安全机制
    1. **自主安全性机制**：存取控制(Access Control)
        + 通过权限在用户之间的传递，使用户自主管理数据库安全性
    2. **强制安全性机制**：
        + 通过对数据和用户强制分类，使得不同类别用户能够访问不同类别的数据
    3. 推断控制机制：
        + 防止通过历史信息，推断出不该被其知道的信息；
        + 防止通过公开信息(通常是一些聚集信息)推断出私密信息(个体信息)，通常在一些由个体数据构成的公共数据库中此问题尤为重要
    4. 数据加密存储机制：
        + 通过加密、解密保护数据，密钥、加密/解密方法与传输
3. DBA的责任和义务
    + 熟悉相关的法规、政策，协助组织的决策者制定好相关的安全策略
    + 规划好安全控制保障措施，例如，系统安全级别、不同级别上的安全控制措施，对安全遭破坏的响应，
    + **划分好数据的安全级别以及用户的安全级别**
    + 实施安全性控制：DBMS专门提供一个DBA账户，该账户是一个超级用户或称系统用户。DBA利用该账户的特权可以进行用户账户的创建以及权限授予和撤消、安全级别控制调整等


### 18. 数据库自主安全性机制
+ 通常情况下，自主安全性是通过授权机制来实现的。
+ 用户在使用数据库前必须由DBA处获得一个账户，并由DBA授予该账户一定的权限，该账户的用户依据其所拥有的权限对数据库进行操作; 同时，该帐户用户也可将其所拥有的权利转授给其他的用户(账户)，由此实现权限在用户之间的传播和控制。
    + 授权者：决定用户权利的人
    + 授权：授予用户访问的权利

1. DBMS自动实现自主安全性：
    + DBMS允许用户定义一些安全性控制规则(用SQL-DCL来定义)
    + 当有DB访问操作时，DBMS自动按照安全性控制规则进行检查，检查通过则允许访问，不通过则不允许访问
2. DBMS将权利和用户(账户)结合在一起，形成一个访问规则表，依据该规则表可以实现对数据库的安全性控制
    + `AccessRule ::=(S, O, t, P)`
        - S: 请求主体(用户)
        - O: 访问对象
        - t: 访问权利
        - P: 谓词
    + { AccessRule｝通常存放在数据字典或称系统目录中，构成了所有用户对DB的访问权利; 
    + 用户多时，可以按用户组建立访问规则
    + 访问对象可大可小(目标粒度Object  granularity):属性/字段、记录/元组、关系、数据库
    + 权利：包括创建、增、删、改、查等
    + 谓词：拥有权利需满足的条件
3. **示例**：员工管理数据库的安全性控制示例`Employee(P#,Pname,Page,Psex,Psalary,D#,HEAD)`
    + 示例要求：
        - 员工管理人员：能访问该数据库的所有内容，便于维护员工信息
        - 收发人员：访问该数据库以确认某员工是哪一个部门的，便于收发工作，只能访问基本信息，其他信息不允许其访问
        - 每个员工：允许其访问关于自己的记录，以便查询自己的工资情况，但不能修改
        - 部门领导：能够查询其所领导部门人员的所有情况
        - 高层领导：能访问该数据库的所有内容，但只能读
    + 两种控制示例
        - 按名控制安全性：存储矩阵
        - 按内容控制安全性：视图
    + 视图是安全性控制的重要手段
    + 通过视图可以限制用户对关系中某些数据项的存取, 例如：
        - 视图1：Create  EmpV1  as  select   *  from  Employee
        - 视图2：Create  EmpV2  as  select  Pname, D#  from Employee
    + 通过视图可将数据访问对象与谓词结合起来，限制用户对关系中某些元组的存取，例如：
        - 视图1： Create  EmpV3  as  select  *  from  Employee  where P# = :UserId
        - 视图2： Create  EmpV4  as  select  *  from  Employee  where Head = :UserId
    + 用户定义视图后，视图便成为一新的数据对象，参与到存储矩阵与能力表中进行描述

#### 18.1 SQL语言的用户与权利
1. SQL语言包含了DDL, DML和DCL。数据库安全性控制是属于DCL范畴
2. 授权机制---自主安全性；视图的运用
3. 关系级别(普通用户) <-- 账户级别(程序员用户) <-- 超级用户(DBA) 
    + (级别1)Select : 读(读DB, Table, Record, Attribute, … )
    + (级别2)Modify : 更新
        - Insert : 插入(插入新元组, … )
        - Update : 更新(更新元组中的某些值, …)
        - Delete : 删除(删除元组, …)
    + (级别3)Create : 创建(创建表空间、模式、表、索引、视图等)
        - Create : 创建
        - Alter : 更新
        - Drop : 删除
4. 级别高的权利自动包含级别低的权利。如某人拥有更新的权利，它也自动拥有读的权利。在有些DBMS中，将级别3的权利称为账户级别的权利，而将级别1和2称为关系级别的权利。
5. 授权命令`GRANT`
```  sql
GRANT {all PRIVILEGES | privilege {,privilege…}}
    ON [TABLE] tablename | viewname
    TO {public | user-id {, user-id…}}
    [WITH GRANT OPTION];
```
    + user-id ，某一个用户账户，由DBA创建的合法账户
    + public, 允许所有有效用户使用授予的权利
    + privilege是下面的权利
        - SELECT | INSERT | UPDATE | DELETE | ALL PRIVILEDGES
    + WITH GRANT OPTION选项是允许被授权者传播这些权利

6. SQL-DCL的控制安全性-授权示例:
    + 假定高级领导为Emp0001, 部门领导为Emp0021, 员工管理员为Emp2001,收发员为Emp5001(均为UserId, 也即员工的P#)
        - Grant All Priviledges ON Employee TO Emp2001;
        - Grant SELECT ON EmpV2 TO Emp5001;
        - Grant SELECT ON EmpV3 TO public;
        - Grant SELECT ON EmpV4 TO Emp0021;
    + 授予视图访问的权利，并不意味着授予基本表访问的权利(两个级别：基本关系级别和视图级别)
    + 授权者授予的权利必须是授权者已经拥有的权利

7. 收回授权命令`REVOKE`
``` sql
REVOKE {all privilEges | priv {, priv…} } 
    ON tablename | viewname
    FROM {public | user {, user…} }; 
```
    + 示例: `revoke select on employee from UserB;`


#### 18.2 自主安全性的授权过程及其问题
##### 18.2.1 授权过程:
1. 第一步：DBA创建DB, 并为每一个用户创建一个账户
    + 假定建立了五个用户：UserA, UserB, UserC, UserD, UserE
2. 第二步：DBA授予某用户账户级别的权利
    + 假定授予UserA
3. 第三步：具有账户级别的用户可以创建基本表或视图, 他也自动成为该表或该视图的属主账户，拥有该表或该视图的所有访问 权利
    + 假定UserA创建了Employee, 则UserA就是Employee表的属主账户
4. 第四步：拥有属主账户的用户可以将其中的一部分权利授予另外的用户，该用户也可将权利进一步授给其他的用户…
    + 假定UserA将读权限授予UserB, 而userB又将其拥有的权限授予UserC,如此将权利不断传递下去。

* 注意授权的传播范围
    + 传播范围包括两个方面：水平传播数量和垂直传播数量
        - 水平传播数量是授权者的再授权用户数目(树的广度)
        - 垂直传播数量是授权者传播给被授权者，再被传播给另一个被授权者, …传播的深度(树的深度)
    + 有些系统提供了传播范围控制，有些系统并没有提供，SQL标准中也并没有限制。
    + 当一个用户的权利被收回时，通过其传播给其他用户的权利也将被收回
    + 如果一个用户从多个用户处获得了授权，则当其中某一个用户收回授权时，该用户可能仍保有权利。例如UserC从UserB和UserE处获得了授权，当UserB收回时，其还将保持UserE赋予其的权利。

##### 18.2.2 强制安全性机制
1. 强制安全性机制
    * 强制安全性通过对数据对象进行安全性分级
        + 绝密(Top Secret), 机密(Secret), 可信(Confidential) 和 无分类(Unclassified)
    * 同时对用户也进行上述的安全性分级
    * 从而强制实现不同级别用户访问不同级别数据的一种机制
2. 强制安全性机制的实现
    * DBMS引入强制安全性机制, 可以通过扩展关系模式来实现
        + 关系模式: R(A1: D1, A2: D2, …, An:Dn)
        + 对属性和元组引入安全性分级特性或称分类特性
            - R(A1: D1, C1, A2: D2, C2…, An:Dn, Cn, TC)其中 C1,C2,…,Cn分别为属性D1,D2,…,Dn的安全分类特性; TC为元组的分类特性
    * 这样, 关系中的每个元组, 都将扩展为带有安全分级的元组
    * 强制安全性机制使得关系形成为多级关系(不同级别用户所能看到的关系的子集)，也出现多重实例、多级关系完整性等许多新的问题或新的处理技巧，在使用中需注意仔细研究。
