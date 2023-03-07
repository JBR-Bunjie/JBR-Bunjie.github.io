### 重新自增：

我们都会遇到这样的问题:SQL Server 数据库原有ID字段,是自增列的,后来把数据全删除后，ID是接着以前的继续增长的.  由于对大量数据的频繁删除，导致ID变的越来越大.影响操作.所以通过以下操作就可实现

```sql
dbcc checkident('TableName',reseed,0)  
```

#### sql 删除表数据并使ID自增重置

方法1：
truncate table 你的表名
//这样不但将数据全部删除，而且重新定位自增的字段

方法2：
delete from 你的表名
dbcc checkident(你的表名,reseed,0) 
//重新定位自增的字段，让它从1开始

方法3：
如果你要保存你的数据，介绍你第三种方法，by QINYI
用phpmyadmin导出数据库，你在里面会有发现哦
编辑sql文件，将其中的自增下一个id号改好，再导入。

---

truncate命令是会把自增的字段还原为从1开始的,或者你试试把table_a清空，然后取消自增，保存，再加回自增，这也是自增段还原为1 的方法。

 

 

\-----------

MySql数据库唯一编号字段（自动编号字段） 
在数据库应用，我们经常要用到唯一编号，以标识记录。在MySQL中可通过数据列的AUTO_INCREMENT属性

来自动生成。MySQL支持多种数据表，每种数据表的自增属性都有差异，这里将介绍各种数据表里的数据

列自增属性。

ISAM表

如果把一个NULL插入到一个AUTO_INCREMENT数据列里去，MySQL将自动生成下一个序列编号。编号从1开

始，并1为基数递增。

把0插入AUTO_INCREMENT数据列的效果与插入NULL值一样。但不建议这样做，还是以插入NULL值为好。

当插入记录时，没有为AUTO_INCREMENT明确指定值，则等同插入NULL值。

当插入记录时，如果为AUTO_INCREMENT数据列明确指定了一个数值，则会出现两种情况，情况一，如果

插入的值与已有的编号重复，则会出现出错信息，因为AUTO_INCREMENT数据列的值必须是唯一的；情况

二，如果插入的值大于已编号的值，则会把该插入到数据列中，并使在下一个编号将从这个新值开始递

增。也就是说，可以跳过一些编号。

如果自增序列的最大值被删除了，则在插入新记录时，该值被重用。

如果用UPDATE命令更新自增列，如果列值与已有的值重复，则会出错。如果大于已有值，则下一个编号

从该值开始递增。

如果用replace命令基于AUTO_INCREMENT数据列里的值来修改数据表里的现有记录，即AUTO_INCREMENT数

据列出现在了replace命令的where子句里，相应的AUTO_INCREMENT值将不会发生变化。但如果replace命

令是通过其它的PRIMARY KEY OR UNIQUE索引来修改现有记录的(即AUTO_INCREMENT数据列没有出现在

replace命令的where子句中)，相应的AUTO_INCREMENT值--如果设置其为NULL(如没有对它赋值)的话--就

会发生变化。

last_insert_id()函数可获得自增列自动生成的最后一个编号。但该函数只与服务器的本次会话过程中

生成的值有关。如果在与服务器的本次会话中尚未生成AUTO_INCREMENT值，则该函数返回0。

其它数据表的自动编号机制都以ISAM表中的机制为基础。

MyISAM数据表

删除最大编号的记录后，该编号不可重用。

可在建表时可用“AUTO_INCREMENT=n”选项来指定一个自增的初始值。

可用alter table table_name AUTO_INCREMENT=n命令来重设自增的起始值。

可使用复合索引在同一个数据表里创建多个相互独立的自增序列，具体做法是这样的：为数据表创建一

个由多个数据列组成的PRIMARY KEY OR UNIQUE索引，并把AUTO_INCREMENT数据列包括在这个索引里作为

它的最后一个数据列。这样，这个复合索引里，前面的那些数据列每构成一种独一无二的组合，最末尾

的AUTO_INCREMENT数据列就会生成一个与该组合相对应的序列编号。
HEAP数据表

HEAP数据表从MySQL4.1开始才允许使用自增列。

自增值可通过CREATE TABLE语句的 AUTO_INCREMENT=n选项来设置。

可通过ALTER TABLE语句的AUTO_INCREMENT=n选项来修改自增始初值。

编号不可重用。

HEAP数据表不支持在一个数据表中使用复合索引来生成多个互不干扰的序列编号。

BDB数据表

不可通过CREATE TABLE OR ALTER TABLE的AUTO_INCREMENT=n选项来改变自增初始值。

可重用编号。

支持在一个数据表里使用复合索引来生成多个互不干扰的序列编号。

InnDB数据表

不可通过CREATE TABLE OR ALTER TABLE的AUTO_INCREMENT=n选项来改变自增初始值。

不可重用编号。

不支持在一个数据表里使用复合索引来生成多个互不干扰的序列编号。

在使用AUTO_INCREMENT时，应注意以下几点：

AUTO_INCREMENT是数据列的一种属性，只适用于整数类型数据列。

设置AUTO_INCREMENT属性的数据列应该是一个正数序列，所以应该把该数据列声明为UNSIGNED，这样序

列的编号个可增加一倍。

AUTO_INCREMENT数据列必须有唯一索引，以避免序号重复。

AUTO_INCREMENT数据列必须具备NOT NULL属性。

AUTO_INCREMENT数据列序号的最大值受该列的数据类型约束，如TINYINT数据列的最大编号是127,如加上

UNSIGNED，则最大为255。一旦达到上限，AUTO_INCREMENT就会失效。

当进行全表删除时，AUTO_INCREMENT会从1重新开始编号。全表删除的意思是发出以下两条语句时：

delete from table_name;
or
truncate table table_name

这是因为进行全表操作时，MySQL实际是做了这样的优化操作：先把数据表里的所有数据和索引删除，然

后重建数据表。如果想删除所有的数据行又想保留序列编号信息，可这样用一个带where的delete命令以

抑制MySQL的优化：

delete from table_name where 1;

这将迫使MySQL为每个删除的数据行都做一次条件表达式的求值操作。

强制MySQL不复用已经使用过的序列值的方法是：另外创建一个专门用来生成AUTO_INCREMENT序列的数据

表，并做到永远不去删除该表的记录。当需要在主数据表里插入一条记录时，先在那个专门生成序号的

表中插入一个NULL值以产生一个编号，然后，在往主数据表里插入数据时，利用LAST_INSERT_ID()函数

取得这个编号，并把它赋值给主表的存放序列的数据列。如：

insert into id set id = NULL;
insert into main set main_id = LAST_INSERT_ID();

可用alter命令给一个数据表增加一个具有AUTO_INCREMENT属性的数据列。MySQL会自动生成所有的编号

。

要重新排列现有的序列编号，最简单的方法是先删除该列，再重建该，MySQL会重新生连续的编号序列。

在不用AUTO_INCREMENT的情况下生成序列，可利用带参数的LAST_INSERT_ID()函数。如果用一个带参数

的LAST_INSERT_ID(expr)去插入或修改一个数据列，紧接着又调用不带参数的LAST_INSERT_ID()函数，

则第二次函数调用返回的就是expr的值。下面演示该方法的具体操作：

先创建一个只有一个数据行的数据表：
create table seq_table (id int unsigned not null);
insert into seq_table values (0);
接着用以下操作检索出序列号：
update seq_table set seq = LAST_INSERT_ID( seq + 1 );
select LAST_INSERT_ID();
通过修改seq+1中的常数值，可生成不同步长的序列，如seq+10可生成步长为10的序列。

该方法可用于计数器，在数据表中插入多行以记录不同的计数值。再配合LAST_INSERT_ID()函数的返回

值生成不同内容的计数值。这种方法的优点是不用事务或LOCK，UNLOCK表就可生成唯一的序列编号。不

会影响其它客户程序的正常表操作。

alter table table_name auto_increment=n;
注意n只能大于已有的auto_increment的整数值,小于的值无效. 
show table status like 'table_name' 可以看到auto_increment这一列是表现有的值.
步进值没法改变.只能通过下面提到last_inset_id()函数变通使用


在使用AUTO_INCREMENT时，应注意以下几点：

AUTO_INCREMENT是数据列的一种属性，只适用于整数类型数据列。

设置AUTO_INCREMENT属性的数据列应该是一个正数序列，所以应该把该数据列声明为UNSIGNED，这样序

列的编号个可增加一倍。

AUTO_INCREMENT数据列必须有唯一索引，以避免序号重复。

AUTO_INCREMENT数据列必须具备NOT NULL属性。

AUTO_INCREMENT数据列序号的最大值受该列的数据类型约束，如TINYINT数据列的最大编号是127,如加上

UNSIGNED，则最大为255。一旦达到上限，AUTO_INCREMENT就会失效。

在不用AUTO_INCREMENT的情况下生成序列，可利用带参数的LAST_INSERT_ID()函数。如果用一个带参数

的LAST_INSERT_ID(expr)去插入或修改一个数据列，紧接着又调用不带参数的LAST_INSERT_ID()函数，

则第二次函数调用返回的就是expr的值。下面演示该方法的具体操作：

先创建一个只有一个数据行的数据表：
create table seq_table (id int unsigned not null);
insert into seq_table values (0);
接着用以下操作检索出序列号：
update seq_table set seq = LAST_INSERT_ID( seq + 1 );
select LAST_INSERT_ID();
通过修改seq+1中的常数值，可生成不同步长的序列，如seq+10可生成步长为10的序列。

该方法可用于计数器，在数据表中插入多行以记录不同的计数值。再配合LAST_INSERT_ID()函数的返回

值生成不同内容的计数值。这种方法的优点是不用事务或LOCK，UNLOCK表就可生成唯一的序列编号。不

会影响其它客户程序的正常表操作。

有两点需要加强注意:
1、只有一列的时候是不行的！
2、自动编号必须作为主键才有效！