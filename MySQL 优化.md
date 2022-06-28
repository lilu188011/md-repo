## MySQL 优化

### 一、实验数据

```sql
#学生表
create table Student(
sid varchar(10),
sname varchar(10),
sage datetime,
ssex varchar(10)
);
 
#教师表
create table Teacher( 
tid varchar(10),
tname varchar(10)
);
 
#科目表
create table Course(
cid varchar(10),
cname varchar(10),
tid varchar(10)
);
 
#成绩表
create table Score( 
sid varchar(10),
cid varchar(10),
score varchar(18.1)
);
insert into Student values('01' , '赵雷' , '1990-01-01' , '男');
insert into Student values('02' , '钱电' , '1990-12-21' , '男');
insert into Student values('03' , '孙风' , '1990-05-20' , '男');
insert into Student values('04' , '李云' , '1990-08-06' , '男');
insert into Student values('05' , '周梅' , '1991-12-01' , '女');
insert into Student values('06' , '吴兰' , '1992-03-01' , '女');
insert into Student values('07' , '郑竹' , '1989-07-01' , '女');
insert into Student values('09' , '张三' , '2017-12-20' , '女');
insert into Student values('10' , '李四' , '2017-12-25' , '女');
insert into Student values('11' , '李四' , '2017-12-30' , '女');
insert into Student values('12' , '赵六' , '2017-01-01' , '女');
insert into Student values('13' , '孙七' , '2018-01-01' , '女');
 
insert into Course values('01' , '语文' , '02'); 
insert into Course values('02' , '数学' , '01'); 
insert into Course values('03' , '英语' , '03');
 
insert into Teacher values('01' , '张三');
insert into Teacher values('02' , '李四'); 
insert into Teacher values('03' , '王五'); 
 
insert into Score values('01' , '01' , 80); 
insert into Score values('01' , '02' , 90); 
insert into Score values('01' , '03' , 99); 
insert into Score values('02' , '01' , 70); 
insert into Score values('02' , '02' , 60); 
insert into Score values('02' , '03' , 80); 
insert into Score values('03' , '01' , 80); 
insert into Score values('03' , '02' , 80); 
insert into Score values('03' , '03' , 80); 
insert into Score values('04' , '01' , 50); 
insert into Score values('04' , '02' , 30); 
insert into Score values('04' , '03' , 20); 
insert into Score values('05' , '01' , 76); 
insert into Score values('05' , '02' , 87); 
insert into Score values('06' , '01' , 31); 
insert into Score values('06' , '03' , 34); 
insert into Score values('07' , '02' , 89); 
insert into Score values('07' , '03' , 98);
```

### SQL优化建议

#### 1、选取最适用的字段属性

```
MySQL可以很好的支持大数据量的存取，但是一般说来，数据库中的表越小，在它上面执行的查询也就会越快。因此，在创建表的时候，为了获得更好的性能，我们可以将表中字段的宽度设得尽可能小。

例如，在定义邮政编码这个字段时，如果将其设置为CHAR(255),显然给数据库增加了不必要的空间，甚至使用VARCHAR这种类型也是多余的，因为CHAR(6)就可以很好的完成任务了。同样的，如果可以的话，我们应该使用MEDIUMINT而不是BIGIN来定义整型字段。

另外一个提高效率的方法是在可能的情况下，应该尽量把字段设置为NOTNULL，这样在将来执行查询的时候，数据库不用去比较NULL值。
对于某些文本字段，例如“省份”或者“性别”，我们可以将它们定义为ENUM类型。因为在MySQL中，ENUM类型被当作数值型数据来处理，而数值型数据被处理起来的速度要比文本类型快得多。这样，我们又可以提高数据库的性能。
```

#### 2、使用连接（JOIN）来代替子查询(Sub-Queries)

```
MySQL从4.1开始支持SQL的子查询。这个技术可以使用SELECT语句来创建一个单列的查询结果，然后把这个结果作为过滤条件用在另一个查询中。例如，我们要将客户基本信息表中没有任何订单的客户删除掉，就可以利用子查询先从销售信息表中将所有发出订单的客户ID取出来，然后将结果传递给主查询，如下所示：
DELETE 
FROM
	customerinfo 
WHERE
	CustomerID NOT IN (
	SELECT
		CustomeID 
FROM
	salesinfo)
	
使用子查询可以一次性的完成很多逻辑上需要多个步骤才能完成的SQL操作，同时也可以避免事务或者表锁死，并且写起来也很容易。但是，有些情况下，子查询可以被更有效率的连接（JOIN）..替代。例如，假设我们要将所有没有订单记录的用户取出来，可以用下面这个查询完成：
SELECT
	* 
FROM
	customerinfo 
WHERE
	CustomerID NOT IN (
	SELECT
		CustomerID 
FROM
	salesinfo)
	
如果使用连接（JOIN）..来完成这个查询工作，速度将会快很多。尤其是当salesinfo表中对CustomerID建有索引的话，性能将会更好，查询如下：
SELECT
	* 
FROM
	customerinfo
	LEFT JOIN salesinfo ON customerinfo.CustomerID = salesinfo.CustomerID 
WHERE
	salesinfo.CustomerID IS NULL
	
连接（JOIN）..之所以更有效率一些，是因为MySQL不需要在内存中创建临时表来完成这个逻辑上的需要两个步骤的查询工作。	
```

#### 3、使用联合(UNION)来代替手动创建的临时表

```
MySQL从4.0的版本开始支持union查询，它可以把需要使用临时表的两条或更多的select查询合并的一个查询中。在客户端的查询会话结束的时候，临时表会被自动删除，从而保证数据库整齐、高效。使用union来创建查询的时候，我们只需要用UNION作为关键字把多个select语句连接起来就可以了，要注意的是所有select语句中的字段数目要想同。下面的例子就演示了一个使用UNION的查询。
SELECT NAME
	,
	Phone 
FROM
	client UNION
SELECT NAME
	,
	BirthDate 
FROM
	author UNION
SELECT NAME
	,
	Supplier 
FROM
	product

```

#### 4、优化的查询语句

```
SELECT
	* 
FROM
	books 
WHERE
	NAME LIKE "MySQL%"
	
如果换用下面的查询，返回的结果一样，但速度就要快上很多：
SELECT
	* 
FROM
	books 
WHERE
	NAME＞ = "MySQL" 
	AND NAME＜ "MySQM"
```

#### 5、索引

##### **1、创建索引**

```
对于查询占主要的应用来说，索引显得尤为重要。很多时候性能问题很简单的就是因为我们忘了添加索引而造成的，或者说没有添加更为有效的索引导致。如果不加索引的话，那么查找任何哪怕只是一条特定的数据都会进行一次全表扫描，如果一张表的数据量很大而符合条件的结果又很少，那么不加索引会引起致命的性能下降。但是也不是什么情况都非得建索引不可，比如性别可能就只有两个值，建索引不仅没什么优势，还会影响到更新速度，这被称为过度索引。
```



##### **2、复合索引**

```
比如有一条语句是这样的：select * from users where area='beijing' and age=22;
如果我们是在area和age上分别创建单个索引的话，由于mysql查询每次只能使用一个索引，所以虽然这样已经相对不做索引时全表扫描提高了很多效率，但是如果在area、age两列上创建复合索引的话将带来更高的效率。如果我们创建了(area, age, salary)的复合索引，那么其实相当于创建了(area,age,salary)、(area,age)、(area)三个索引，这被称为最佳左前缀特性。因此我们在创建复合索引时应该将最常用作限制条件的列放在最左边，依次递减。
```

##### **3、索引不会包含有NULL值的列**

```
只要列中包含有NULL值都将不会被包含在索引中，复合索引中只要有一列含有NULL值，那么这一列对于此复合索引就是无效的。所以我们在数据库设计时不要让字段的默认值为NULL。
```

##### **4、使用短索引**

```
对串列进行索引，如果可能应该指定一个前缀长度。例如，如果有一个CHAR(255)的 列，如果在前10 个或20 个字符内，多数值是惟一的，那么就不要对整个列进行索引。短索引不仅可以提高查询速度而且可以节省磁盘空间和I/O操作。
```

##### **5、排序的索引问题**

mysql查询只使用一个索引，因此如果where子句中已经使用了索引的话，那么order by中的列是不会使用索引的。因此数据库默认排序可以符合要求的情况下不要使用排序操作；尽量不要包含多个列的排序，如果需要最好给这些列创建复合索引。

##### **6、like语句操作**

```
一般情况下不鼓励使用like操作，如果非使用不可，如何使用也是一个问题。like “%aaa%” 不会使用索引而like “aaa%”可以使用索引。
```

##### **7、不要在列上进行运算**

```
select * from users where YEAR(adddate)<2007;
将在每个行上进行运算，这将导致索引失效而进行全表扫描，因此我们可以改成
select * from users where adddate<‘2007-01-01';
```

##### **8、不使用NOT IN和<>操作**

```
NOT IN和<>操作都不会使用索引将进行全表扫描。NOT IN可以NOT EXISTS代替，id<>3则可使用id>3 or id<3来代替。
```

##### 9、**性能状态关键指标**

```
 QPS，Queries Per Second：每秒查询数，一台数据库每秒能够处理的查询次数

  TPS，Transactions Per Second：每秒处理事务数

  通过show status查看运行状态，会有300多条状态信息记录，其中有几个值帮可以我们计算出QPS和TPS，如下：

  Uptime：服务器已经运行的实际，单位秒

  Questions：已经发送给数据库查询数

  Com_select：查询次数，实际操作数据库的

  Com_insert：插入次数

  Com_delete：删除次数

  Com_update：更新次数

  Com_commit：事务次数

  Com_rollback：回滚次数

  那么，计算方法来了，基于Questions计算出QPS：

  mysql> show global status like 'Questions';

  mysql> show global status like 'Uptime';

  QPS = Questions / Uptime

  基于Com_commit和Com_rollback计算出TPS：

  mysql> show global status like 'Com_commit';

  mysql> show global status like 'Com_rollback';

  mysql> show global status like 'Uptime';

  TPS = (Com_commit + Com_rollback) / Uptime

  另一计算方式：基于Com_select、Com_insert、Com_delete、Com_update计算出QPS

  mysql> show global status where Variable_name in('com_select','com_insert','com_delete','com_update');

  等待1秒再执行，获取间隔差值，第二次每个变量值减去第一次对应的变量值，就是QPS

  TPS计算方法：

  mysql> show global status where Variable_name in('com_insert','com_delete','com_update');

  计算TPS，就不算查询操作了，计算出插入、删除、更新四个值即可。

  经网友对这两个计算方式的测试得出，当数据库中myisam表比较多时，使用Questions计算比较准确。当数据库中innodb表比较多时，则以Com_*计算比较准确。

  5.2 开启慢查询日志

  MySQL开启慢查询日志，分析出哪条SQL语句比较慢，使用set设置变量，重启服务失效，可以在my.cnf添加参数永久生效。

mysql> set global slow-query-log=on  #开启慢查询功能

mysql> set global slow_query_log_file='/var/log/mysql/mysql-slow.log';  #指定慢查询日志文件位置

mysql> set global log_queries_not_using_indexes=on;   #记录没有使用索引的查询

mysql> set global long_query_time=1;   #只记录处理时间1s以上的慢查询

分析慢查询日志，可以使用MySQL自带的mysqldumpslow工具，分析的日志较为简单。

 # mysqldumpslow -t 3 /var/log/mysql/mysql-slow.log    #查看最慢的前三个查询

也可以使用percona公司的pt-query-digest工具，日志分析功能全面，可分析slow log、binlog、general log。

  分析慢查询日志：pt-query-digest /var/log/mysql/mysql-slow.log

  分析binlog日志：mysqlbinlog mysql-bin.000001 >mysql-bin.000001.sql 

  pt-query-digest --type=binlog mysql-bin.000001.sql 

  分析普通日志：pt-query-digest --type=genlog localhost.log
```

### SQL排查 ###

- 慢查询日志
----------
	show variables like '%slow_query_log%';
	临时开启 
	set global slow_query_log = 1;
	
	永久开启
	show_query_log=1
	show_query_log_file=/var/lib/mysql/localhost-slow.log	
	阀值查看
	show variables like '%long_query_time%';
	
	临时开启 
		set global long_query_time = 1;
		
	查询慢查询sql总数
		show global status like '%slow_queries%'


- mysqlddumpslow(查看慢查询日志)
-----------
	通过mysqlddumpslow 查看日志
		-s, 是表示按照何种方式排序
		    c：访问计数
		    l：锁定时间
		    r:返回记录
		    al：平均锁定时间
		    ar：平均访问记录数
		    at：平均查询时间
	 
		-t, 是top n的意思，即为返回前面多少条的数据；
		-g, 后边可以写一个正则匹配模式，大小写不敏感的；
	
	-- 获取返回记录最多的3个sql
		mysqlddumpslow -s -r -t 3 log文件
	
	-- 获取访问次数最多的3个sql 
		mysqlddumpslow -s c -t 3 log文件
	
	--按照时间排序，前10条包含left join查询语句的sql
	
		mysqlddumpslow -s t -t 10 -g  "left join" log文件

- mysql5.7(虚拟列)
-----------
	建表时添加虚拟列
		`SimpleDate_dayofweek` tinyint(4) GENERATED ALWAYS AS 
		(dayofweek(SimpleDate)) VIRTUAL
	
	为表添加虚拟列
		alter table user add user_name varchar(20) generated always as 
		(data->'$.name');	



- mysql binlog
----------
	binlog格式
		–基于SQL语句的复制(statement-based replication,SBR)， 
		–基于行的复制(row-based replication,RBR)， 
		–混合模式复制(mixed-based replication,MBR)。	
	
	1）查看binlog_format
		show variables like 'binlog_format'
	
	2）查看是否开启binlog
		show variables like 'log_bin'
	
	3）获取binlog文件列表
		show binary logs
	
	4）查看当前正在写入的binlog文件
		show master status
	
	5） 查看master上的binlog
		show master logs
	
	6）只查看第一个binlog文件的内容
		show binlog events
	
	7）查看指定binlog文件的内容
		show binlog events in 'mysql-bin.000002'
	
	8） 删除binlog
		reset master;//删除master的binlog
		reset slave;    //删除slave的中继日志
		purge master logs before '2012-03-30 17:20:00';
		//删除指定日期以前的日志索引中binlog日志文件
		purge master logs to 'mysql-bin.000002'; //删除指定日志文件的日志索引中binlog日志文件
	9）解析binlog 
		mysql-binlog-connector-java

- 查看锁表
----------
	查看锁表情况
		show status  like '%lock%';
	
	查看正在被锁定的的表
		show OPEN TABLES where In_use > 0;
		
	查看正在执行进程	
		show PROCESSLIST;
	
	kill进程
		kill id
	
	了解索引的效果
		show status like "Handler_read%"
		Handler_read_key 值高表示索引效果好，Handler_read_rnd_next值高表示索引低效。 

- 查看mysql语句运行时间
----------
	show profiles
	查看是否开启		show variables like "%pro%"
	设置开启			set profiling = 1
	可以开始执行一些想要分析的sql语句了，执行完后，show profiles；
	show profile for query 1  即可查看第1个sql语句的执行的各个操作的耗时详情


​	
- mysql锁方面
----------
	select * from information_schema.innodb_trx;
	
	select * from information_schema.INNODB_LOCKS;
	
	select * from information_schema.INNODB_LOCK_WAITS;
	
	select
		r.trx_id waiting_trx_id,
		r.trx_mysql_thread_id waiting_thread,
		r.trx_query waiting_query,
		b.trx_id blocking_trx_id,
		b.trx_mysql_thread_id blocking_thread,
		b.trx_query blocking_query
	from information_schema.innodb_lock_waits w
	inner join information_schema.innodb_trx b
	on b.trx_id = w.blocking_trx_id
	inner join information_schema.innodb_trx r
	on r.trx_id = w.requesting_trx_id;
- ### **explian用法解析**

  1. ### 用法简介

     - 使用

       ```mysql
       EXPLAIN SELECT * FROM t1
       ```

     - explain可以分析结果

       1. 表的读取顺序
       2. 数据读取操作的操作类型
       3. 哪些索引可以使用
       4. 哪些索引被实际使用
       5. 表之间的引用
       6. 每张表有多少行被优化器查询

     2. ### 执行计划各字段含义

        1. #### ID

           1. id相同，执行顺序由上至下 
           2. id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行
           3. id相同不同，同时存在   相同从上往下 id越大越先执行

        2. #### select_type

           - SIMPLE 	   简单的select查询，查询中不包含子查询或者UNION
           - PRIMARY      查询中若包含任何复杂的子部分，最外层查询则被标记为PRIMARY
           - SUBQUERY   在SELECT或WHERE列表中包含了子查询
           - DERIVED       在FROM列表中包含的子查询被标记为DERIVED（衍生）， MySQL会递归执行这些        子查询，把结果放在临时表中
           - UNION          若第二个SELECT出现在UNION之后，则被标记为UNION：若UNION包含在FROM子句的子查询中，外层SELECT将被标记为：DERIVED
           - UNION RESULT 从UNION表获取结果的SELECT

           

        3. #### table

           - 当前执行的表

        4. #### type

           | type类型 | 语义                                                         |
           | -------- | ------------------------------------------------------------ |
           | system   | 表只有一行记录（等于系统表），这是const类型的特列，平时不会出现，这个也可以忽略不计 |
           | const    | 表示通过索引一次就找到了，const用于比较primary key 或者unique索引。因为只匹配一行数据，所以很快。如将主键置于where列表中，MySQL就能将该查询转换为一个常量。 |
           | eq_ref   | 唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描 |
           | ref      | 非唯一性索引扫描，返回匹配某个单独值的所有行，本质上也是一种索引访问，它返回所有匹配某个单独值的行，然而，它可能会找到多个符合条件的行，所以他应该属于查找和扫描的混合体。 |
           | range    | 只检索给定范围的行，使用一个索引来选择行，key列显示使用了哪个索引，一般就是在你的where语句中出现between、< 、>、in等的查询，这种范围扫描索引比全表扫描要好，因为它只需要开始于索引的某一点，而结束于另一点，不用扫描全部索引。 |
           | index    | index类型只遍历索引树                                        |
           | all      | 全表扫描                                                     |

        5. #### possible_keys 和 key

           | `possible_keys` | 显示可能应用在这张表中的索引，一个或多个。查询涉及到的字段上若存在索引，则该索引将被列出，**但不一定被查询实际使用**。 |
           | --------------- | ------------------------------------------------------------ |
           | key             | 实际使用的索引，如果为NULL，则没有使用索引。（可能原因包括没有建立索引或索引失效） |

        6. #### key_len

           ```
           表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度，在不损失精确性的情况下，长度越短越好。key_len显示的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的。
   ```
        
		  key_len 计算方法
		 utf-8  一个字符两个字节
		 null 	一个字节标识
 varchar() 可变  两个字节标识
        
   ```
        
7. #### ref
        
           ```
           显示索引的那一列被使用了，如果可能的话，最好是一个常数。哪些列或常量被用于查找索引列上的值。 
   ```
        
8. #### rows
        
           ```
           根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数，也就是说，用的越少越好 
   ```
        
9. #### Extra
        
           ```
           包含不适合在其他列中显式但十分重要的额外信息
   ```
        
   1. ##### Using filesort（九死一生）
        
           ```
              说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取。MySQL中无法利用索引完成的排序操作称为“文件排序”。
      ```
        

        ​      

           2. ##### Using temporary（十死无生）
        
              ```
      使用了用临时表保存中间结果，MySQL在对查询结果排序时使用临时表。常见于排序order by和分组查询group by。 
              ```

              

           3. ##### Using index（发财了）
        
              ```
      表示相应的select操作中使用了覆盖索引（Covering Index），避免访问了表的数据行，效率不错。如果同时出现using where，表明索引被用来执行索引键值的查找；如果没有同时出现using where，表明索引用来读取数据而非执行查找动作。
              ```

              

           4. ##### Using where
        
              ```
      表明使用了where过滤
              ```

           5. ##### Using join buffer
        
              ```
      表明使用了连接缓存,比如说在查询的时候，多表join的次数非常多，那么将配置文件中的缓冲区的join buffer调大一些。
              ```

              

           6. ##### impossible where
        
              ```
      where子句的值总是false，不能用来获取任何元组
              ```

              

           7. ##### select tables optimized away
        
              ```
      在没有GROUPBY子句的情况下，基于索引优化MIN/MAX操作或者对于MyISAM存储引擎优化COUNT(*)操作，不必等到执行阶段再进行计算，查询执行计划生成的阶段即完成优化。
              ```

              

           8. ##### distinct
        
              ```
      优化distinct操作，在找到第一匹配的元组后即停止找同样值的动作
              ```

              

           9. join语句优化总结

              - 尽可能减少join语句中的NestedLoop的循环总次数：“永远用小结果集驱动大的结果集”。

              - 优先优化NestedLoop的内层循环

              - 保证join语句中被驱动表上Join条件字段已经被索引

              - 当无法保证被驱动表的JOIN条件字段被索引且内存资源充足的前提下，不要太吝啬JoinBuffer的设置。
        
                