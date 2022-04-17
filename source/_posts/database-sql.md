---
title: 「数据库」嵌入式SQL语言
date: 2018-09-01 14:32:28
tags: [数据库, 后端开发]
categories: 数据库
---

### 概述
* 交互式SQL语言有很多优点：记录集合操作、非过程性操作、一条语句就可实现复杂查询的结果，
* 然而，交互式SQL本身也有很多局限：<!-- more -->
    + 从使用者角度：专业人员可熟练写出SQL语句，但大部分的普通用户并非可以
    + 从SQL本身角度：特别复杂的检索结果难以用一条交互式SQL语句完成，此时需要结合高级语言中经常出现的顺序、分支和循环结构来帮助处理
* 因此，高级语言+SQL语言：
    + 既继承高级语言的过程控制性
    + 又结合SQL语言的复杂结果集操作的非过程性
    + 同时又为数据库操作者提供安全可靠的操作方式：通过应用程序进行操作
* 嵌入式SQL语言
    + 将SQL语言嵌入到某一种高级语言中使用
    + 这种高级语言，如C/C++, Java, PowerBuilder等，又称宿主语言(Host Language)
    + 嵌入在宿主语言中的SQL与前面介绍的交互式SQL有一些不同的操作方式


### 1. 变量声明与数据库连接
1. 以宿主语言**C语言**为例，对比交互式SQL语言与嵌入式SQL语言
    + 交互式SQL:`select Sname, Sage from Student where Sname='张三';`
    + 嵌入式SQL:`exec sql select Sname, Sage into :vSname, :vSage from Student where Sname='张三';`
2. 典型特点
    + **exec sql**引导SQL语句: 提供给C编译器，以便对SQL语句预编译成C编译器可识别的语句
    + 增加一 **into子句**: 该子句用于指出接收SQL语句检索结果的程序变量
    + 由冒号引导的**程序变量**,如: ‘:vSname’, ‘:vSage’

#### 1.1 变量的声明与使用
* 在嵌入式SQL语句中可以出现宿主语言语句所使用的变量，这些变量需要特殊的声明：
``` sql
exec sql begin declare section;
    char vSname[10], specName[10]="张三";
    int vSage;
exec sql end declare section;
```
* 变量声明和赋值中，要注意：
    + 宿主程序的字符串变量长度应比字符型字段的长度多1个。因宿主程序的字符串尾部多一个终止符为'\0'，而程序中用双引号来描述。
    + 宿主程序变量类型与数据库字段类型之间有些是有差异的, 有些DBMS可支持自动转换，有些不能。
* 声明的变量，可以在宿主程序中赋值，然后传递给SQL语句的where等子句中，以使SQL语句能够按照指定的要求(可变化的)进行检索。
* 嵌入式比交互式SQL语句灵活了一些：只需改一下变量值，SQL语句便可反复使用，以检索出不同结果。
* 示例：
``` sql
exec sql begin declare section;
    char vSname[10], specName[10]="张三";
    int vSage;
exec sql end declare section;
//用户可在此处基于键盘输入给specName赋值
exec sql select Sname, Sage into :vSname, :vSage from Student where Sname = :specName;
//比较相应的交互式SQL语句：
select Sname, Sage from Student where Sname = '张三';
```


#### 1.2 程序与数据库的连接和断开
##### 1.2.1 数据库的连接connect
在嵌入式SQL程序执行之前，首先要与数据库进行连接, 不同DBMS，具体连接语句的语法略有差异
1. SQL标准中建议的连接语法为：
    + `exec sql connect to target-server as connect-name user user-name;`
    + 或 `exec sql connect to default;`
2. Oracle中数据库连接:
    + `exec sql connect  :user_name identified by :user_pwd;`
3. DB2 UDB中数据库连接:
    + `exec sql connect  to  mydb user  :user_name using  :user_pwd;`

##### 1.2.1 数据库的断开disconnect
在嵌入式SQL程序执行之后，需要与数据库断开连接
1. SQL标准中建议的断开连接的语法为：
    + `exec sql disconnect connect-name;`
    + 或 `exec sql disconnect current;`
2. Oracle中断开连接:
    + `exec sql commit release;`
    + 或 `exec sql rollback release;`
3. DB2 UDB中断开连接:
    + `exec sql connect reset;`
    + `exec sql disconnect current;`

#### 1.3 SQL执行的提交与撤消
SQL语句在执行过程中，必须有提交和撤消语句才能确认其操作结果
1. SQL执行的提交：
    + `exec  sql commit  work;`
2. SQL执行的撤消：
    + `exec  sql rollback  work;`
3. 为此，很多DBMS都设计了捆绑提交/撤消与断开连接在一起的语句,以保证在断开连接之前使用户确认提交或撤消先前的工作，例如Oracle中：
    + `exec  sql commit  release;`
    + 或 `exec  sql rollback  release;`


### 2. 事务Transaction
1. 从应用程序员角度：事务是一个存取或改变数据库内容的程序的一次执行，或者说一条或多条SQL语句的一次执行被看作一个事务
2. 从微观角度，或者从DBMS角度：事务是数据库管理系统提供的控制数据操作的一种手段，通过这一手段，应用程序员将一系列的数据库操作组合在一起作为一个整体进行操作和控制，以便数据库管理系统能够提供一致性状态转换的保证。
3. 简单来说：事务是作为单个逻辑工作单元执行的一系列操作；多个操作作为一个整体向系统提交，要么都执行，要么都不执行；**事务是一个不可分割的工作逻辑单元**。

#### 2.1 事务的特性: ACID
1. **原子性**Atomicity : DBMS能够保证事务的一组更新操作是原子不可分的，即对DB而言，要么都执行，要么都不执行
2. **一致性**Consistency: DBMS保证事务的操作状态是正确的，符合一致性的操作规则，它是进一步由隔离性来保证的
3. **隔离性**Isolation: DBMS保证并发执行的多个事务之间互相不受影响。例如两个事务T1和T2, 即使并发执行，也相当于或者先执行了T1,再执行T2;或者先执行了T2, 再执行T1。
4. **持久性**Durability: DBMS保证已提交事务的影响是持久的，被撤销事务的影响是可恢复的。

> 换句话说：具有ACID特性的若干数据库基本操作的组合体被称为事务。


### 3. 数据集与游标
读取单行结果处理与多行结果处理的差异：Into子句与游标(Cursor)
1. 检索单行结果，可将结果直接传送到宿主程序的变量中(Into)
    + 示例：`exec sql select Sname,Sage into :vSname,:vSage from Student where Sname = :specName;`
2. 检索多行结果，则需使用游标(Cursor)
    + 游标是指向某检索记录集的指针
    + 通过这个指针的移动，每次读一行，处理一行，再读一行… , 直至处理完毕
    + 读一行操作是通过Fetch…into语句实现的：每一次Fetch, 都是先向下移动指针，然后再读取
    + 记录集有结束标识EOF, 用来标记后面已没有记录了
* 游标(Cursor)的使用需要先定义、再打开(执行)、接着一条接一条处理，最后再关闭
* 游标可以定义一次，多次打开(多次执行)，多次关闭

#### 3.1 游标的使用方法
1. Cursor的定义：declare cursor
``` sql
EXEC SQL DECLARE cursor_name CURSOR FOR
    Subquery
    [ORDER BY result_column [ASC | DESC][, result_column …]
    [FOR [ READ ONLY | UPDATE [OF columnname [, columnname…]]]];
//示例:
exec sql declare cur_student cursor for
    select Sno, Sname, Sclass from Student where Sclass= :vClass
    order by Sno
    for read only ;
```

2. Cursor的打开和关闭：open cursor //close cursor
    + EXEC SQL OPEN cursor_name;
    + EXEC SQL CLOSE cursor_name;

3. Cursor的数据读取：Fetch
``` sql
EXEC SQL FETCH cursor_name
    INTO host-variable , [host-variable, …];
//示例:
exec sql declare cur_student cursor for
    select Sno, Sname, Sclass from Student where Sclass= :vClass
    order by Sno for read only ;
exec sql open cur_student;
…
exec sql fetch cur_student into :vSno, :vSname, :vSage
…
exec sql close cur_student;
```

#### 3.2 可滚动游标
1. ODBC支持的可滚动Cursor
    + 标准的游标始终是自开始向结束方向移动的，每fetch一次，向结束方向移动一次；一条记录只能被访问一次；再次访问该记录只能关闭游标后重新打开
    + ODBC( Open DataBase Connectivity)是一种跨DBMS的DB操作平台，它在应用程序与实际的DBMS之间提供了一种通用接口
    + 许多实际的DBMS并不支持可滚动游标，但通过ODBC可以使用该功能
2. 可滚动游标是可使游标指针在记录集之间灵活移动、使每条记录可以反复被访问的一种游标
    + 可滚动游标移动时需判断是否到结束位置，或到起始位置
        - 可通过判断是否到EOF位置(最后一条记录的后面), 或BOF位置(起始记录的前面)
        - 如果不需区分，可通过whenever  not found语句设置来检测

``` sql
EXEC SQL DECLARE cursor_name [INSENSITIVE] [SCROLL] CURSOR
[WITH HOLD] FOR Subquery
[ORDER BY result_column [ASC | DESC][, result_column …]
[FOR READ ONLY | FOR UPDATE OF columnname [,
columnname ]…];
EXEC SQL FETCH
[ NEXT | PRIOR | FIRST | LAST
| [ABSOLUTE | RELATIVE] value_spec ]
FROM cursor_name INTO host-variable [, host-variable …];
```
* `NEXT`向结束方向移动一条；
* `PRIOR`向开始方向移动一条；
* `FIRST`回到第一条；
* `LAST`移动到最后一条；
* `ABSOLUT  value_spec`定向检索指定位置的行, value_spec由1至当前记录集最大值；
* `RELATIVE  value_spec`相对当前记录向前或向后移动，value_spec为正数向结束方向移动，为负数向开始方向移动

#### 3.3 数据库记录的增删改
1. 数据库记录的删除
    + 一种是查找删除(与交互式DELETE语句相同)，一种是定位删除

``` sql
EXEC SQL DELETE FROM tablename [corr_name]
    WHERE search_condition | WHERE CURRENT OF cursor_name;
//示例：查找删除
exec sql delete from customers c where c.city = ‘Harbin’ and
    not exists ( select * from orders o where o.cid = c.cid);
//示例：定位删除
exec sql declare delcust cursor for
    select cid from customers c where c.city =‘harbin’ and
    not exists ( select * from orders o where o.cid = c.cid)
    for update of cid;
exec sql open delcust
While (TRUE) {
    exec sql fetch delcust into :cust_id;
    exec sql delete from customers where current of delcust ; }
```

2. 数据库记录的更新
    + 一种是查找更新(与交互式Update语句相同)，一种是定位更新

``` sql
EXEC SQL UPDATE tablename [corr_name]
    SET columnname = expr [, columnname = expr …]
    [ WHERE search_condition ] | WHERE CURRENT OF cursor_name;
//示例：查找更新
exec sql update student s set sclass = ‘035102’
    where s.sclass = ‘034101’
// 示例：定位更新
exec sql declare stud cursor for
    select * from student s where s.sclass =‘034101’
    for update of sclass;
exec sql open stud
While (TRUE) {
    exec sql fetch stud into :vSno, :vSname, :vSclass;
    exec sql update student set sclass = ‘035102’ where current of stud ; }
```

3. 数据库记录的插入
    + 只有一种类型的插入语句

``` sql
EXEC SQL INSERT INTO tablename [ (columnname [,columnname, …] )]
    [ VALUES (expr [ , expr , …] ) | subqurey ] ;
//示例：插入语句
exec sql insert into student ( sno, sname, sclass)
    values (‘03510128’, ‘张三’, ‘035101’) ;
//示例：插入语句
exec sql insert into masterstudent ( sno, sname, sclass)
    select sno, sname, sclass from student;
```


### 4. 状态捕获及错误处理机制
#### 4.1 基本机制
* 状态，是嵌入式SQL语句的执行状态，尤其指一些出错状态；有时程序需要知道这些状态并对这些状态进行处理
* 嵌入式 SQL程序中，状态捕获及处理有三部分构成
    1. 设置SQL通信区: 一般在嵌入式SQL程序的开始处便设置
        + `exec sql include sqlca;`
    2. 设置状态捕获语句: 在嵌入式SQL程序的任何位置都可设置；可多次设置；但有作用域
        + `exec sql whenever sqlerror goto report_error;`
    3. 状态处理语句: 某一段程序以应对SQL操作的某种状态
        + `report_error: exec sql rollback;`
* SQL通信区: SQLCA
    1. SQLCA是一个已被声明过的具C语言的结构形式的内存信息区，其中的成员变量用来记录SQL语句执行的状态，便于宿主程序读取与处理
    2. SQLCA是DBMS(执行SQL语句)与宿主程序之间交流的桥梁之一
* 状态捕获语句: `exec sql whenever condition action;`
    + Whenever语句的作用是设置一个“条件陷阱”, 该条语句会对其后面的所有由Exec SQL语句所引起的对数据库系统的调用自动检查它是否满足条件(由condition指出).
        - SQLERROR: 检测是否有SQL语句出错。其具体意义依赖于特定的DBMS
        - NOT FOUND: 执行某一SQL语句后，没有相应的结果记录出现
        - SQLWARNING: 不是错误，但应引起注意的条件
    + 如果满足condition, 则要采取一些动作(由action指出)
        - CONTINUE: 忽略条件或错误，继续执行
        - GOTO 标号: 转移到标号所指示的语句，去进行相应的处理
        - STOP: 终止程序运行、撤消当前的工作、断开数据库的连接
        - DO函数或 CALL函数: 调用宿主程序的函数进行处理，函数返回后从引发该condition的Exec SQL语句之后的语句继续进行
* 状态捕获语句Whenever的作用范围是其后的所有Exec SQL语句，一直到程序中出现另一条相同条件的Whenever语句为止，后面的将覆盖前面的。
``` SQL
int main() {
    exec sql whenever sqlerror stop;
    … …
    goto s1
    … …
    exec sql whenever sqlerror continue;
    s1: exec sql update agents set percent = percent + 1;
    … …
}
//S1标号指示的语句受第二个Whenever语句约束。
//注意：作用域是语句在程序中的位置，而不是控制流程(因是预编译程序处理条件陷阱)
```

* 状态捕获语句Whenever的使用容易引发无限循环
``` SQL
int main() {
    exec sql whenever sqlerror goto handle_error;
    exec sql create table customers(cid char(4) not null,
    cname varchar(13), … … );
    … …
    handle_error:
        exec sql whenever sqlerror continue;// 控制是否无限循环：无，则可能；有，则不会
        exec sql drop customers;
        exec sql disconnect;
        fprintf(stderr,”could not create customers table\n”);
        return -1;
}
```

#### 4.2 状态信息
典型DBMS系统记录状态信息的三种方法
* 状态记录:
    1. `sqlcode`: 典型DBMS都提供一个sqlcode变量来记录其执行sql语句的状态，但不同DBMS定义的sqlcode值所代表的状态意义可能是不同的。
        + sqlcode== 0, successful call;
        + sqlcode < 0, error, e.g., from connect, database does not exist , –16;
        + sqlcode > 0, warning, e.g., no rows retrieved from fetch
    2. `sqlca.sqlcode`: 支持SQLCA的产品一般要在SQLCA中填写sqlcode来记录上述信息; 除此而外，sqlca还有其他状态信息的记录
    3. `sqlstate`: 有些DBMS提供的记录状态信息的变量是sqlstate或sqlca.sqlstate
* 当我们不需明确知道错误类型，而只需知道发生错误与否，则我们只要使用前述的状态捕获语句即可，而无需关心状态记录变量(隐式状态处理)
* 但我们程序中如要自行处理不同状态信息时，则需要知道以上信息，但也需知道正确的操作方法(显式状态处理)


#### 4.3 程序自身进行错误信息的处理
正确的显式状态处理示例:
``` SQL
exec sql begin declar section;
    char SQLSTATE[6];
exec sql end declare section;
exec sql whenever sqlerror goto handle_error;
… …
exec sql whenever sqlerror continue;
exec sql create table custs
    (cid char(4) not null, cname varchar(13), … … );
if (strcmp(SQLSTATE, “82100”)==0)
    <处理82100错误的程序>
    … …
```
上述的if 语句是能被执行的，因为create table发生错误时是继续向下执行的。


### 5. 动态SQL
#### 5.1 动态SQL的概念
动态SQL是相对于静态SQL而言的
* 静态SQL特点：SQL语句在程序中已经按要求写好，只需要把一些参数通过变量(高级语言程序语句中不带冒号) 传送给嵌入式SQL语句即可(嵌入式SQL语句中带冒号)
* 动态SQL特点：SQL语句可以在程序中动态构造，形成一个字符串，然后再交给DBMS执行，交给DBMS执行时仍旧可以传递变量

#### 5.2 动态SQL的两种执行方式
如SQL语句已经被构造在host-variable字符串变量中, 则：
1. **立即执行语句**: 运行时编译并执行
    + `EXEC SQL EXECUTE IMMEDIATE :host-variable;`
2. **Prepare-Execute-Using语句**: PREPARE语句先编译，编译后的SQL语句允许动态参数，EXECUTE语句执行，用USING语句将动态参数值传送给编译好的SQL语句
    + `EXEC SQL PREPARE sql_temp FROM :host-variable;`
    + `EXEC SQL EXECUTE sql_temp USING :cond-variable`


### 6. 数据字典与SQLDA
#### 6.1 数据字典的概念
数据字典(Data dictionary)，又称为系统目录(System Catalogs)
* 是系统维护的一些表或视图的集合，这些表或视图存储了数据库中各类对象的定义信息，这些对象包括用Create语句定义的表、列、索引、视图、权限、约束等, 这些信息又称数据库的元数据--关于数据的数据。
* 不同DBMS术语不一样：数据字典(Data Dictionary(Oracle))、目录表(DB2 UDB)、系统目录(INFORMIX)、系统视图(X/Open)
* 不同DBMS中系统目录存储方式可能是不同的, 但会有一些信息对DBA公开。这些公开的信息, DBA可以使用一些特殊的SQL命令来检索。

#### 6.2 数据字典的内容构成
数据字典通常存储的是数据库和表的元数据，即模式本身的信息：
1. 与关系相关的信息
    + 关系名字
    + 每一个关系的属性名及其类型
    + 视图的名字及其定义
    + 完整性约束
2. 用户与账户信息，包括密码
3. 统计与描述性数据：如每个关系中元组的数目
4. 物理文件组织信息：
    + 关系是如何存储的(顺序/无序/散列等)
    + 关系的物理位置
5. 索引相关的信息

#### 6.3 数据字典的结构
1. 也是存储在磁盘上的关系
2. 专为内存高效访问设计的特定的数据结构

* 可能的字典数据结构
    + `Relation_metadata` = `(relation_name, number_of_attributes, storage_organization, location)`
    + `Attribute_metadata` = `(attribute_name, relation_name, domain_type, position, length)`
    + `User_metadata` = `(user_name, encrypted_password, group)`
    + `Index_metadata` = `(index_name, relation_name, index_type, index_attributes)`
    + `View_metadata` = `(view_name, definition)`

#### 6.4 X/Open标准的系统目录
1. X/Open标准中有一个目录表Info_Schem.Tables, 该表中的一行是一个已经定义的表的有关信息
    + `Table_Schem`：表的模式名(通常是表所有者的用户名)
    + `Table_Name`：表名
    + `Table_Type`：`'Base_Table'`或`'View'`
2. 可以使用SQL语句来访问这个表中的信息，比如了解已经定义了哪些表，可如下进行：
    + `Select Table_Name From Tables;`
3. 模式的含义是指某一用户所设计和使用的表、索引及其他与数据库有关的对象的集合，因此表的完整名应是：模式名.表名。这样做可允许不同用户使用相同的表名，而不混淆。
4. 一般而言，一个用户有一个模式。可以使用Create Schema语句来创建模式(用法参见相关文献)，在Create Table等语句可以使用所定义的模式名称。

#### 6.5 Oracle的数据字典
1. Oracle数据字典由视图组成，分为三种不同形式，由不同的前缀标识
    + `USER_` :用户视图，用户所拥有的对象，在用户模式中
    + `ALL_`  :扩展的用户视图，用户可访问的对象
    + `DBA_`  :DBA视图(所有用户都可访问的DBA对象的子集)
2. Oracle数据字典中定义了三个视图`USER_Tables`, `ALL_Tables`, 和`DBA_Tables`供DBA和用户使用数据字典中关于**表的信息**
3. 同样, Oracle数据字典中也定义了三个视图`USER_TAB_Columns`, `ALL_TAB_Columns`(`Accessible_Columns`), 和`DBA_TAB_Columns`供DBA和用户使用数据字典中关于表的**列的信息**
4. 可以使用SQL语句来访问这些表中的信息：
    + `Select Column_Name From ALL_TAB_Columns Where Table_Name = ‘STUDENT’;`
5. Oracle数据字典中还定义了其他视图
    + `TABLE_PRIVILEDGE`(或`ALL_TAB_GRANTS`)
    + `COLUMN_PRIVILEDGE`(或`ALL_COL_GRANTS`)可访问表的权限，列的权限
    + `CONSTRAINT_DEFS`(或`ALL_CONSTRAINTS`)可访问表的各种约束
6. 可以使用下述命令获取Oracle定义的所有视图信息
    + `Select view_name from all_views where owner = ‘SYS’ and view_name like ‘ALL_%’ or view_name like ‘USER_%’;`
7. 如果用户使用Oracle,可使用其提供的`SQL*PLUS`进行交互式访问
8. 动态SQL: 表和列都已知，动态构造检索条件。
9. 动态SQL:检索条件可动态构造，表和列也可动态构造。

#### 6.6 SQLDA
构造复杂的动态SQL需要了解数据字典及SQLDA，已获知关系模式信息
1. SQLDA: SQL Descriptor Area, SQL描述符区域。
    + SQLDA是一个内存数据结构，内可装载关系模式的定义信息，如列的数目，每一列的名字和类型等等
    + 通过读取SQLDA信息可以进行更为复杂的动态SQL的处理
    + 不同DBMS提供的SQLDA格式并不是一致的。


### 7. ODBC简介
#### 7.1 ODBC定义
ODBC：Open DataBase Connection，ODBC是一种标准---不同语言的应用程序与不同数据库服务器之间通讯的标准。
* 一组API(应用程序接口)，支持应用程序与数据库服务器的交互
* 应用程序通过调用ODBC API, 实现
    1. 与数据服务器的连接
    2. 向数据库服务器发送SQL命令
    3. 一条一条的提取数据库检索结果中的元组传递给应用程序的变量
* 具体的DBMS提供一套驱动程序，即Driver库函数，供ODBC调用，以便实现数据库与应用程序的连接。
* ODBC可以配合很多高级语言来使用，如C,C++, C#, Visual Basic, PowerBuilder等等；

#### 7.2 通过ODBC连接数据库
1. ODBC应用前，需要确认具体DBMS Driver被安装到ODBC环境中
2. 当应用程序调用ODBC API时，ODBC API会调用具体DBMS Driver库函数，DBMS Driver库函数则与数据库服务器通讯，执行相应的请求动作并返回检索结果
3. ODBC应用程序首先要分配一个SQL环境，再产生一个数据库连接句柄
4. 应用程序使用SQLConnect()，打开一个数据库连接，SQLConnect()的具体参数:
    + `connection handle`, 连接句柄
    + `the server`，要连接的数据库服务器
    + `the user identifier`，用户
    + `password` ，密码
    + `SQL_NTS` 类型说明前面的参数是空终止的字符串
5. 示例
``` sql
int ODBCexample(){
    RETCODE error; /* 返回状态吗 */
    HENV env; /* 环境变量 */
    HDBC conn; /* 连接句柄 */
    SQLAllocEnv(&env);
    SQLAllocConnect(env, &conn);
    //分配数据库连接环境
    SQLConnect(conn, "aura.bell-labs.com", SQL_NTS, "avi", SQL_NTS, avipasswd", SQL_NTS);
    //打开一个数据库连接
    { …. Do actual work … }
    //与数据库通讯
    SQLDisconnect(conn);
    SQLFreeConnect(conn);
    SQLFreeEnv(env);
    //断开连接与释放环境
}
```

#### 7.3 通过ODBC与数据库服务器进行通讯
1. 应用程序使用SQLExecDirect()向数据库发送SQL命令；
2. 使用SQLFetch()获取产生的结果元组；
3. 使用SQLBindCol()绑定C语言变量与结果中的属性
    + 当获取一个元组时，属性值会自动地传送到相应的C语言变量中
4. SQLBindCol()的参数：
    + ODBC定义的stmt变量, 查询结果中的属性位置
    + SQL到C的类型变换, 变量的地址. 
    + 对于类似字符数组一样的可变长度类型，应给出
        - •变量的最大长度
        - •当获取到一个元组后，实际长度的存储位置.
        - •注: 当返回实际长度为负数，说明是一个空值。
5. 示例
``` sql
char branchname[80]; float balance;
int lenOut1, lenOut2;
HSTMT stmt;
SQLAllocStmt(conn, &stmt);
//分配一个与指定数据库连接的新的语句句柄
char * sqlquery = "select branch_name, sum (balance)
    from account
    group by branch_name";
error = SQLExecDirect(stmt, sqlquery, SQL_NTS);
//执行查询，stmt句柄指向结果集合
if (error == SQL_SUCCESS) {
SQLBindCol(stmt, 1, SQL_C_CHAR, branchname , 80, &lenOut1);
SQLBindCol(stmt, 2, SQL_C_FLOAT, &balance, 0 , &lenOut2);
//绑定高级语言变量与stmt句柄中的属性
while (SQLFetch(stmt) >= SQL_SUCCESS) {
//提取一条记录，结果数据被存入高级语言变量中
    printf (" %s %g\n", branchname, balance);
    }
}
SQLFreeStmt(stmt, SQL_DROP);
//释放语句句柄
```


#### 7.4 ODBC的其他功能
1. 动态SQL语句的预编译-动态参数传递功能
2. 获取元数据特性
    + 发现数据库中的所有关系的特性 以及
    + 发现每一个查询结果的列的名字和类型等；
3. 默认, 每一条SQL语句都被作为一个独立的能够自动提交的事务来处理。
    + 应用程序可以关闭一个连接的自动提交特性
        - `SQLSetConnectOption(conn, SQL_AUTOCOMMIT, 0)}`
    + 此时事务要显式地给出提交和撤销的命令
        - `SQLTransact(conn, SQL_COMMIT)` or `SQLTransact(conn, SQL_ROLLBACK)`


### 8. JDBC简介
#### 8.1 JDBC定义
JDBC：Java DataBase Connection，JDBC是一组Java版的应用程序接口API，提供了Java应用程序与数据库服务器的连接和通讯能力。
* JDBC API 分成两个程序包：
    + Java.sql 核心API --J2SE(Java2标准版)的一部分。使用`java.sql.DriverManager`类、`java.sql.Driver`和`java.sql.Connection`接口连接到数据库
    + Javax.sql 可选扩展API--J2EE(Java2企业版)的一部分。包含了基于`JNDI(Java Naming and Directory Interface, Java命名和目录接口)`的资源，以及管理连接池、分布式事务等，使用DataSource接口连接到数据库。

#### 8.2 JDBC的功能
1. `java.sql.DriverManager`——处理驱动的调入并且对产生新数据库连接提供支持
2. `Java.sql.Driver`——通过驱动进行数据库访问，连接到数据库的应用程序必须具备该数据库的特定驱动。
3. `java.sql.Connection`——代表对特定数据库的连接。
4. `Try {…} Catch {…}` ——异常捕获及其处理
5. `java.sql.Statement`——对特定的数据库执行SQL语句
6. `java.sql.PreparedStatement` —— 用于执行预编译的SQL语句
7. `java.sql.CallableStatement` ——用于执行对数据库内嵌过程的调用。
8. `java.sql.ResultSet`——从当前执行的SQL语句中返回结果数据。

#### 8.3 使用JDBC API访问数据库的过程
1. 概念性的基本过程
    + 打开一个连接；创建“Statement”对象，并设置查询语句；使用Statement对象执行查询，发送查询给数据库服务器和返回结果给应用程序；处理错误的例外机制
2. 具体实施过程
    1. •传递一个Driver给DriverManager，加载数据库驱动。
        + `Class.forName()`
    2. •通过URL得到一个Connection对象, 建立数据库连接
        + `DriverManager.getConnection(sDBUrl)`
        + `DriverManager.getConnection(sDBUrl,sDBUserID,sDBPassword)`
    3. •接着创建一个Statement对象(PreparedStatement或CallableStatement)，用来查询或者修改数据库。
        + `Statement stmt=con.createStatement()`
    4. •查询返回一个ResultSet。
        + `ResultSet rs=stmt.executeQuery(sSQL)`
3. 示例：
``` java
public static void JDBCexample(String dbid, String userid, String passwd)
{ try { //错误捕获
    Class.forName ("oracle.jdbc.driver.OracleDriver");
    Connection conn = DriverManager.getConnection(
        "jdbc:oracle:thin:@db.yale.edu:1521:univdb", userid, passwd);
    //加载数据库驱动，建立数据库连接
    Statement stmt = conn.createStatement();
    //创建一个语句对象
    … Do Actual Work ….
    //进行SQL语句的执行与处理工作
    stmt.close();
    conn.close();
    //关闭语句对象，关闭连接
} catch (SQLException sqle) {
    System.out.println("SQLException : " + sqle); }
}
```

4. 完整的示例程序
``` java
public static void JDBCexample(String dbid, String userid, String passwd)
{ try {
    Class.forName ("oracle.jdbc.driver.OracleDriver");
    Connection conn = DriverManager.getConnection(
        "jdbc:oracle:thin:@db.yale.edu:1521:univdb", userid, passwd);
    Statement stmt = conn.createStatement();
    try {
        stmt.executeUpdate( "insert into instructor values
        (‘77987', ‘Kim', ‘Physics’,98000)");
    } catch (SQLException sqle) {
        System.out.println("插入错误:" + sqle);
    }
    ResultSet rset = stmt.executeQuery(
        "select dept_name, avg(salary) from instructor group by dept_name");
    while ( rset.next() ) {
        System.out.println(rset.getString(“dept_name") + " " + rset.getFloat(2));
    }
    stmt.close();
    conn.close();
} catch (SQLException sqle) {
    System.out.println("SQLException:" + sqle);
}
}
```


### 9. 嵌入式SQL-ODBC-JDBC三者比较
执行一条SQL语句，读取执行的结果集合
1. 嵌入式SQL的思维模式
    1. 建立数据库连接
    2. 声明一个游标
    3. 打开游标
    4. 读取一条记录(循环)
    5. 关闭游标
    6. 断开数据库连接
2. ODBC的思维模式
    1. 建立数据库连接
    2. 分配语句句柄
    3. 用句柄执行SQL
    4. 建立高级语言变量与句柄属性的对应
    5. 读取一条记录(循环)
    6. 释放语句句柄
    7. 断开数据库连接
3. JDBC的思维模式
    1. 建立数据库连接
    2. 创建语句对象
    3. 用语句对象执行SQL，并返回结果对象
    4. 从结果对象获取一条记录
    5. 提取对象的属性值传给高级语言变量(返回上一步)
    6. 释放语句对象
    7. 断开数据库连接

* 相同点: 都是建立数据库连接, 执行sql, 处理结果, 释放连接, 流程基本一致
* 不同点, 操作方式的不同:
    + 嵌入式SQL按照语句进行操作
    + ODBC按照函数来进行操作
    + JDBC按照对象来进行操作
