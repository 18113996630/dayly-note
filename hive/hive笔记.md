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

## Hive查询相关

> https://cwiki.apache.org/confluence/display/Hive/LanguageManual

### SELECT部分

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

