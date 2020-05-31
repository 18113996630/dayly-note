## 安装

1. 下载压缩包

   > https://hive.apache.org/downloads.html

2. 下载MySQL驱动

3. 安装MySQL

4. 安装Hive

   1. 配置**hive-site.xml**

      ```xml
      <configuration>
          <property>
             <name>javax.jdo.option.ConnectionURL</name>
             <value>jdbc:mysql://n151:3306/hive?createDatabaseIfNotExist=true&amp;useSSL=false</value>
          </property>
          <property>
             <name>javax.jdo.option.ConnectionDriverName</name>
             <value>com.mysql.jdbc.Driver</value>
          </property>
          <property>
             <name>javax.jdo.option.ConnectionUserName</name>
             <value>root</value>
          </property>
          <property>
             <name>javax.jdo.option.ConnectionPassword</name>
             <value>123456</value>
          </property>
      </configuration>
      ```

   2. 将MySQL驱动包放到Hive的**lib**目录下

   3. 配置环境变量

      ```xml
      export HIVE_HOME=/usr/local/hive
      export PATH=$HIVE_HOME/bin:$PATH
      ```

5. 初始化Hive元数据

   ```shell
   /usr/local/hive/bin/schematool -dbType mysql -initSchema
   ```

6. 启动Hive

   ```shell
   /usr/local/hvie/bin/hive
   ```

7. hive远程连接

   ```shell
   /usr/local/hive/bin/hiveserver2 &
   
   客户端使用jdbc方式连接，端口为10000，无特殊配置下可以不用输账号密码
   ```

## Hive常见配置

> https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties

```xml
配置文件位置：~/.hiverc（hive启动时自动加载）

set hive.cli.print.current.db=true;
set hive.cli.print.header=true;
set hive.mapred.mode=nonstrict;

set hive.exec.dynamic.partition.mode=nonstrict;
-- 设置hive首先选取本地模式进行数据查询
set hive.exec.mode.local.auto=true;

-- 设置hive在使用group by时在map端进行聚合，但是需要更多的内存
set hive.map.aggr=true;

-- 增加自己的jar包，创建自定义的udf
add jar hdfs://n151:9000/user/hadoop/extend/udf.jar;
create temporary function parse_area as 'com.hrong.hive.udf.CalculateArea';
```

- set hive.mapred.mode=nonstrict

  >Default Value: 
  >
  >- Hive 0.x: `nonstrict`
  >- Hive 1.x: `nonstrict`
  >- Hive 2.x: `strict` 
  >
  >Hive操作正在执行的模式。在`strict` 模式下，不允许运行某些有风险的查询。比如不能进行全表扫描、使用Order By 必须进行limit操作

## 数据类型

- tinyint		1byte有符号整数
- smalint		2byte有符号整数
- int			4byte有符号整数
- bigint		8byte有符号整数
- boolean		是否
- float		单精度浮点数
- double		双精度浮点数
- string		字符串
- timestamp	整数浮点数或者字符串
- binary		字节数组
- struct	   struct <street:string,city:string>   {"street":"mozijie","city":"cd"}
- map		 map<string,float>					{"s1":110.4,"s2":50.1}
- array		array<string>						["emp1","emp2","emp3"]

## 表操作

- 创建外部分区表

  ```sql
  create external TABLE emp(id int,name string) partitioned by(month string) row format delimited fields terminated by ' ' stored as textfile;
  ```

- 修改表

  - 修改表名

    ```sql
    ALTER TABLE emp rename to [employee];
    ```

  - 新增分区并指定数据位置

    ```sql
    ALTER TABLE emp add partition(month='9') location '/data/hive/emp/month=9/';
    ```

  - 删除分区：	ALTER TABLE emp drop partition(month='9');如果是外部表，使用msck repair TABLE empty;可以恢复分区信息

  - 修改分区表指定分区的数据来源

    ```sql
    ALTER TABLE emp partition(month='10') set location '/data/hive/emp/month=10/emp10';
    ```

  - 修改列信息

    ```sql
    ALTER TABLE emp CHANGE COLUMN name mingzi STRING;
    ```

  - 增加列信息

    ```sql
    ALTER TABLE temp add COLUMNS (xing STRING COMMENT 'gender',salary double);
    ```

  - 修改存储格式

    ```sql
    ALTER TABLE employee partition(month='12') set FILEFORMAT sequencefile;
    ```

- 删除表

  drop TABLE fjs;	如果表名不存在不报错
  如果是外部表，则只会删除表的元信息，hdfs上的数据不会删除

- 快速复制一个分区表

  ```sql
  原分区表：emp
  1. create TABLE [employee]  like emp;
  2. dfs -cp /user/hive/warehouse/school.db/emp/* /user/hive/warehouse/school.db/employee/;
  3. msck repair TABLE [employee] ;
  4. show partitions [employee] ;
  ```

## 数据操作

- 将查询的数据插入表格

  ```sql
  <inesrt overwrite == inesrt into>
  	
  insert overwrite table employee partition(month=9) select id,name from emp where month=9;
  
  exception： FAILED: SemanticException [Error 10044]: Line 1:23 Cannot insert into target table because column number/types are different '10': Table insclause-0 has 2 columns, but query has 3 columns.
  	
  note：select 后面不能加上分区列，因为hive在插入的时候会默认插入已经指定的month字段，所以在查询的时候需要将分区字段去掉
  	
  根据内容进行动态分区：insert overwrite table emp partition(age) select id,name,age from emp_n;  
  会根据最后一个字段(age)进行产生分区信息，即使列名不一样也不影响
  	
  将指定分区的数据加载入目标表
  insert overwrite table stu partition(classid=1,teacherid) select id,name ,teacherid from stu_tmp where classid=1;    
  如果分区信息指定了的话在select的映射区则不能出现该字段。静态分区的信息应该在动态分区的前面。
  ```

- 将查询的数据保存到本地

  ```sql
  1、hive> insert [overwrite] local directory '/usr/data/hive' select * from mydb.stu;   -- hive的保存方式<缺点：列值之间无分割符>
  	
  2、$> hive -e "select * from mydb.stu" > a.txt
  	
  3、$> hive -f "/usr/data/hive/hql/save.hql' > a.txt
  ```

- 使用SQL向表中插入数据

  ```sql
  insert into page_ads select 'page-one', array(1,2,3);
  ```

  

## Hive查询相关

> https://cwiki.apache.org/confluence/display/Hive/LanguageManual

### 1. SELECT部分

#### 1、Multi-Group-By Inserts

> https://cwiki.apache.org/confluence/display/Hive/LanguageManual+GroupBy

- 使用场景

  > 对一个表进行多次group by，并将结果写入到其他表或者是保存到hdfs

- 使用方法

  ```sql
  -- s为源表，将两次group by的结果分别保存到tmp1、temp2
  -- 注意 select 语句中不能再写from了
  from s 
  insert overwrite table temp1 
  	select classid, avg(math) group by class_id 
  insert overwrite table temp2 
  	select classid, math, avg(english)  group by class_id, math;
  ```

#### 2、Order, Sort, Cluster, and Distribute By

> https://cwiki.apache.org/confluence/display/Hive/LanguageManual+SortBy

##### 	1、order by

> 1. 如果在`strict `模式下，使用order by必须加`limit`
>
>    **原因：**使用了`order by`之后，会有一个`reducer`进行结果排序，如果数据量比较大，则会需要很长时间
>
> 2. 如果`order by`列中存在`null`值，且排序方式为`asc`时，`null`值会被排在**最前面**
> 3. 在`Hive 3.0.0`以后的版本，如果在子查询中使用了`order by`但是没有加`limit`的话，则优化器会将`order by`操作进行删除
> 4. `order by`作用时机为`reduce`的时候，所以能保证**最终结果的有序性**

##### 2、sort by

> 1. `sort by`的时机为`reduce`之前，作用与`order by`类似，但是只能保证**分区内数据的有序性**。如果`sort by`作用的列为数字类型的，则根据**数字**进行排序，否则根据**字典顺序**进行排序
> 2. 在`Hive 3.0.0`以后的版本，如果在子查询中使用了`sort by`但是没有加`limit`的话，则优化器会将`sort by`操作进行删除

##### 3、distribute by

> 1. `distribute`为分发的意思，即根据`dsitribute by`的列进行分发，相同值的数据一定在同一个`reducer`上

##### 	4、 cluster by

> 1. `cluster by` = `distribute by` + `sort by`，先根据列A进行reduce，再根据列B进行sort
> 2. 如果`reduce`和`sort`的列为同一列，则可以使用`cluster by`

#### 3、LanguageManual Transform

> https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Transform

#### 4、Hive Operators and User-Defined Functions (UDFs)

#### 5、join

> https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Joins

- 语法

  > ```
  > join_table:
  >  table_reference [INNER] JOIN table_factor [join_condition]
  >  table_reference {LEFT|RIGHT|FULL} [OUTER] JOIN table_reference join_condition
  >  table_reference LEFT SEMI JOIN table_reference join_condition
  >  table_reference CROSS JOIN table_reference [join_condition] (as of Hive ``0.10``)
  > ```

> 1. Hive 0.13.0之后可以使用隐式join，即：`SELECT * FROM table1 t1, table2 t2 WHERE t1.id = t2.id`，默认为`inner join`
>
> 2. 多个表`join`时，如果用作连接的列为同一列，则最终只会转化为一个`map-reduce`作业
>
> 3. 多个表`join`时，除了最后一个join的表之外的表会缓存到buffer中，所以，将数据量最大的表放在最后可以有效减少内存的使用量
>
> 4. 对于表的筛选条件，有两种写法
>
>    1. select * from a left join b on a.id=b.a_id where a.id=**
>    2. select * from a left join b on a.id=b.a_id and a.id=**
>
>    如果是分区表，则可能出现这种情况：
>    
>    ​	存在a中的id在b表中找不到对应的a_id，left join仍会将数据查询出来，那么通过a.id进行筛选则会筛选出b中分区列为空的数据

```sql
select * from class;
-- 查询结果：
class.id	class.name
1				one
2				two
3				three
-----------------------------------------------------------------------------
select * from student;
-- 查询结果：
student.id	student.name	student.class_id
1				s1					1
2				s2					2
3				s3					2
4				s4					2
5				s5					4
-----------------------------------------------------------------------------
select * from class t1, student t2 where t1.id=t2.class_id;
=select * from class t1 join student t2 on t1.id=t2.class_id;
=select * from class t1 inner join student t2 on t1.id=t2.class_id;
-- 查询结果：
t1.id	t1.name	 t2.id	t2.name		t2.class_id
2			two		1		s1			1
2			two		2		s2			2
2			two		3		s3			2
2			two		4		s4			2

-----------------------------------------------------------------------------
select * from class t1 left join student t2 on t1.id=t2.class_id;
-- 查询结果：
t1.id	t1.name	t2.id	t2.name	t2.class_id
1		one		1		  s1	   1
2		two		2		  s2	   2
2		two		3		  s3	   2
2		two		4		  s4	   2
NULL	NULL	5		  s5	   4

-----------------------------------------------------------------------------
select * from class t1 right join student t2 on t1.id=t2.class_id;
-- 查询结果：
t1.id	t1.name	t2.id	t2.name	t2.class_id
1		 one	 1		  s1	    1
2		 two	 2		  s2	    2
2		 two	 3		  s3	    2
2		 two	 4		  s4	    2
3		 three	 NULL	  NULL	    NULL
-----------------------------------------------------------------------------
select * from class t1 full join student t2 on t1.id=t2.class_id;
-- 查询结果：
t1.id	t1.name	t2.id	t2.name	t2.class_id
1		one		1		s1			1
2		two		4		s4			2
2		two		3		s3			2
2		two		2		s2			2
3		three	NULL	NULL		NULL
NULL	NULL	5		s5			4
```

### 2. lateral

> https://cwiki.apache.org/confluence/display/Hive/LanguageManual+LateralView
>
> ```sql
> lateralView: LATERAL VIEW udtf(expression) tableAlias AS columnAlias (``','` `columnAlias)*
> fromClause: FROM baseTable (lateralView)*
> ```
>
> lateral view与用户定义的表生成功能（例如explode（））结合使用。如内置表生成函数中所述，UDTF为每个输入行生成零个或多个输出行。侧视图首先将UDTF应用于基本表的每一行，然后将结果输出行与输入行连接起来以形成具有所提供表别名的虚拟表。
>
> 可以使用多个lateral view，按照先后顺序加载，靠后的lateral可以使用靠前的lateral

```sql
page_ads.page	page_ads.ids
page-one	[1,2,3]
page-two	[4,2,5]

select id, count(1) as cnt from page_ads lateral view explode(ids) tmp as id group by id order by cnt desc;

-- 查询结果
id		cnt
2		2
5		1
4		1
3		1
1		1
```

## 常见查询

### 两次分组

```sql
set hive.exec.dynamic.partition.mode=nonstrict;
set hive.exec.mode.local.auto=true;

drop table group_twice;
create external table group_twice
(
    id      int,
    name    string,
    cost    int,
    income int
) partitioned by (dt string) stored as sequencefile;


insert into group_twice partition (dt)
VALUES (1, '张三', 10, 100, '20200528'),
       (2, '李四', 5, 50, '20200528'),
       (3, '张三', 15, 85, '20200528'),
       (4, '李四', 8, 42, '20200529'),
       (5, '张三', 10, 75, '20200529'),
       (6, '李四', 20, 22, '20200529'),
       (7, '张三', 50, 125, '20200530'),
       (8, '李四', 12, 10, '20200530');
id  name       cost 	income 		dt       
1	张三    	10		100		20200528
2	李四    	5		50		20200528
3	张三    	15		85		20200528
4	李四    	8		42		20200529
5	张三    	10		75		20200529
6	李四    	20		22		20200529
7	张三    	50		125		20200530
8	李四    	12		10		20200530


select name, dt, sum(cost) as total_cost,
       max(sum(income)) over (partition by name order by sum(income) desc) as most_income
from group_twice
where dt between '20200528' and '20200530'
group by dt, name
order by name, dt;

name       dt 	total_cost 	   most_income       
张三	20200528	25	       185
张三	20200529	10	       185
张三	20200530	50	       185
李四	20200528	5	       64
李四	20200529	28	       64
李四	20200530	12	       64


```

