## 1、CASE表达式

**要点：**

1. 在使用了GROUP BY的时候，通过case可以根据分组后的某个列值进行转化

2. 使用聚合函数与case可以简单的进行行转列

3. 可以写列名的地方基本上都可以使用case

### 使用

- 简单case表达式

  ```sql
  CASE sex
  	WHEN 1 THEN '男'
  	WHEN 2 THEN '女'
  ELSE '其他' END
  ```

- 搜索case表达式

  ```sql
  CASE WHEN sex = 1 THEN '男'
  		 WHEN sex = 2 THEN '女'
  ELSE '其他' END
  ```

- 注意
  - 各分支返回的数值类型相同
  - 记得写ELSE和END

### 例1 统计区域人数

****

```sql
select * from PopTbl;

city population							
乐山		100
成都		400
昆明		200
武汉		200
贵阳		300

------------
----目标：---
------------
province	  total
其它			 400
四川省		    500
贵州省		    300
```

```
select 
case when city='成都' then '四川省'
		 when city='乐山' then '四川省'
		 when city='贵阳' then '贵州省'
else '其它' end as province,
sum(population) as total 
from PopTbl 
# group by 后面按照SQL规范应该是放置case when语句段，mysql可以直接放置alias
group by province;
```

### 例2 查看每个地区的男女人数

****

```sql
select * from poptbl2;
city   sex 	population
乐山	  1		20
乐山	  2		80
成都	  1		250
成都	  2		150
深圳	  1		100
深圳	  2		100

------------
----目标：---
------------
city 	 man	 woman
乐山		20		80
成都		250		150
深圳		100		100
```

```
select
    t.city,
    # sum用于对分组后数据的求和
    sum(case when t.sex='1' then t.population else 0 end) as man,
    sum(case when t.sex='2' then t.population else 0 end) as women
from poptbl2 t GROUP BY t.city;
```

### 例3 salary大于等于30万降薪10%，25-28万之间的涨薪20%

**update时存在条件分支**

```sql
select * from salaries;
name  salary
孙六	290000
张三	220000
李四	324000
王五	280000
```

```sql
update salaries set salary = 
case when salary >= 300000 then 0.9*salary
		 when salary BETWEEN 250000 and 280000 then 1.2*salary 
else salary end;
```

### 例4 **调换表中的值(如果是调换主键值则需要先去掉主键约束)**

****

```sql
select * from class;

id  name
1	一班
3	三班
2	二班

------------
----目标：---
------------

id  name
1	二班
3	三班
2	一班
```

```sql
update class set name = 
case when name = '一班' then '二班'
		 when name = '二班' then '一班'
else name end; 
```

### 例5 **根据coursemaster和opencourses表查看每个月开设的课程信息**

```sql
select * from coursemaster;

course_id	course_name
1			会计入门
2			财务知识
3			簿记考试
4			税务师

select * from opencourses;
month		course_id
202001			1
202001			4
202002			2
202002			3
202002			4
202003			4


------------
----目标：---
------------

course_name		2020-01	2020-02	2020-03
会计入门			√		×		×
财务知识			×		√		×
簿记考试			×		√		×
税务师		 	 	 √	 	 √	 	 √
```

```sql
# 使用exists
select 
    t.course_name,
    case when EXISTS (select 1 from opencourses o where t.course_id=o.course_id and o.month = 202001) then '√' else '×' end as '2020-01',
    case when EXISTS (select 1 from opencourses o where t.course_id=o.course_id and o.month = 202002) then '√' else '×' end as '2020-02',
    case when EXISTS (select 1 from opencourses o where t.course_id=o.course_id and o.month = 202003) then '√' else '×' end as '2020-03'
from coursemaster t;

# 使用in
select 
	t.course_name,
    case when t.course_id in (select o.course_id from opencourses o where  o.month = 202001) then '√' else '×' end as '2020-01',
    case when t.course_id in (select o.course_id from opencourses o where o.month = 202002) then '√' else '×' end as '2020-02',
    case when t.course_id in (select o.course_id from opencourses o where o.month = 202003) then '√' else '×' end as '2020-03'
from coursemaster t;
```

### 例6 获取只加入了一个社团的学生的社团ID和加入了多个社团的学生的主社团ID

```sql
select * from studentclub;

std_id	 club_id		club_name	main_club_flg
100			1			跆拳道			Y
100			2			管弦乐			N
200			2			管弦乐			N
200			3			羽毛球			Y
200			4			足球			 N
300			4			足球			 N
400			5			游泳			 N
500			6			围棋			 N

------------
----目标：---
------------

std_id	 club_id
100			1
200			3
300			4
400			5
500			6
```

```sql
# union两次处理结果
select std_id, max(club_id) from studentclub t group by std_id having count(1)=1
union all 
select std_id, club_id from studentclub t where t.main_club_flg='Y';

# case when
select 
	t.std_id,
	case when count(1)=1 then max(t.club_id)
	else max(case when t.main_club_flg='Y' then t.club_id else null end) end as 'club_id' 
from studentclub t GROUP BY t.std_id;

```

### 练习题1

> 查出每个人在语文、数学两科中的最高分
>
> 查出每个人在语文、数学、英语三科中的最高分

```sql
select * from scores;

name	chinese		math	english
A			1		 2			3
B			5		 5			2
C			4		 7			1
D			3		 3			8

------------
----目标：---
------------
# 语文、数学两科中的最高分
name	greatest
A			2
B			5
C			7
D			3

# 语文、数学、英语三科中的最高分
name	greatest
A			3
B			5
C			7
D			8

```

```sql
# 语文、数学两科中的最高分
select name, 
case when chinese>math then chinese else math end as greatest
from scores;

# 语文、数学、英语三科中的最高分
-- 使用case when
select name, 
case when 
	case when chinese>english then chinese else english end < math 
then 
	math	
else 
	case when chinese>english then chinese else english end
end as greatest
from scores;

-- 使用列转行，再group by
select name, max(score) as greatest from(
    select name, chinese as score from scores
    union all 
    select name, math as score from scores
    union all 
    select name, english as score from scores
) tmp group by name;

```

### 练习题2

```sql
select * from poptbl2;

city	sex		population
乐山	 1			20
乐山	 2			80
成都	 1			250
成都	 2			150
深圳	 1			100
深圳	 2			100

------------
----目标：---
------------

性别		总计	  四川省	广州省
男		 370	 270	 100
女		 330	 230	 100
```

```sql
select 
	case when sex='1' then '男' else '女' end as 性别,
	sum(population) as 总计,
	sum(case when city='乐山' or city='成都' then population else null end) as 四川省,
	sum(case when city='深圳' then population else null end) as 广州省
from poptbl2 group by sex;
```



##  2、自连接

### 例题

### 1、实现rank函数

```
select * from products;

name	price
柠檬		30
橘子		100
苹果		50
葡萄		50
西瓜		80
香蕉		50

------------
----目标：---
------------

name	price	rank
橘子	100		  	1
西瓜	80		  	2
香蕉	50		  	3
苹果	50		  	3
葡萄	50		  	3
柠檬	30		  	6
```

```sql
# 使用子查询实现rank（要进行排名，意思就是根据某个字段找出自身更大的元素的数量）
select
	(select count(1) from products p1 where p1.price>p.price)+1 as rank,
	name,
	price
from products p order by rank;

# 使用left join将表进行自连接，连接条件on为非等值连接，可进行rank排序
select 
	count(p2.name)+1 as rank, 
	p1.name, 
	max(p1.price) as price  # 进行了group by之后取每组的最大值
from products p1 left join products p2 
	on p1.price<p2.price 
group by p1.name 
order by rank;

# 内连接和外连接的差别（内连接只查询出能连接上的数据，left join则以左表为标准）

select * from products p1, products p2 where p1.price < p2.price ORDER BY p1.price;

select * from products p1 left join products p2 on p1.price < p2.price ORDER BY p1.price;
```

**自连接要点：**

- 自连接经常和**非等值连接**结合使用
- 自连接的性能开销大，应尽量**为用于连接的列建立索引**

### 练习题

### 1、查出分组rank信息

```sql
CREATE TABLE district_products
(district  VARCHAR(16) NOT NULL,
 name      VARCHAR(16) NOT NULL,
 price     INTEGER NOT NULL,
 PRIMARY KEY(district, name, price));

INSERT INTO district_products VALUES('东北', '橘子',	100);
INSERT INTO district_products VALUES('东北', '苹果',	50);
INSERT INTO district_products VALUES('东北', '葡萄',	50);
INSERT INTO district_products VALUES('东北', '柠檬',	30);
INSERT INTO district_products VALUES('关东', '柠檬',	100);
INSERT INTO district_products VALUES('关东', '菠萝',	100);
INSERT INTO district_products VALUES('关东', '苹果',	100);
INSERT INTO district_products VALUES('关东', '葡萄',	70);
INSERT INTO district_products VALUES('关西', '柠檬',	70);
INSERT INTO district_products VALUES('关西', '西瓜',	30);
INSERT INTO district_products VALUES('关西', '苹果',	20);

select * from district_products;

district name	price
东北		柠檬	 30
东北		橘子	 100
东北		苹果	 50
东北		葡萄	 50
关东		柠檬	 100
关东		苹果	 100
关东		菠萝	 100
关东		葡萄	 70
关西		柠檬	 70
关西		苹果	 20
关西		西瓜	 30

------------
----目标：---
------------

district name price	   rank
东北		橘子	100		  1
东北		苹果	50		  2
东北		葡萄	50		  2
东北		柠檬	30		  4
关东		柠檬	100		  1
关东		菠萝	100		  1
关东		苹果	100		  1
关东		葡萄	70		  4
关西		柠檬	70		  1
关西		西瓜	30		  2
关西		苹果	20		  3
```

```sql
# 子查询
select 
	d1.district,
	d1.name,
	d1.price,
	(select count(1) from district_products d2 where d1.district=d2.district and d1.price < d2.price)+1 as rank
from district_products d1 ORDER BY d1.district, rank;

# 自连接
select 
	d1.district, 
	d1.name, 
	max(d1.price) as price, 
	count(d2.name)+1 as rank
from district_products d1 left outer join district_products d2 
	on d1.district = d2.district 
	and d1.price < d2.price
group by d1.district, d1.name order by d1.district, rank;
```

### 2、更新ranking字段

```sql
CREATE TABLE district_products2
(district  VARCHAR(16) NOT NULL,
 name      VARCHAR(16) NOT NULL,
 price     INTEGER NOT NULL,
 ranking   INTEGER,
 PRIMARY KEY(district, name));

INSERT INTO district_products2 VALUES('东北', '橘子',	100, NULL);
INSERT INTO district_products2 VALUES('东北', '苹果',	50 , NULL);
INSERT INTO district_products2 VALUES('东北', '葡萄',	50 , NULL);
INSERT INTO district_products2 VALUES('东北', '柠檬',	30 , NULL);
INSERT INTO district_products2 VALUES('关东', '柠檬',	100, NULL);
INSERT INTO district_products2 VALUES('关东', '菠萝',	100, NULL);
INSERT INTO district_products2 VALUES('关东', '苹果',	100, NULL);
INSERT INTO district_products2 VALUES('关东', '葡萄',	70 , NULL);
INSERT INTO district_products2 VALUES('关西', '柠檬',	70 , NULL);
INSERT INTO district_products2 VALUES('关西', '西瓜',	30 , NULL);
INSERT INTO district_products2 VALUES('关西', '苹果',	20 , NULL);

select * from district_products2 ORDER BY district, ranking;

district	 name	  price	     ranking
东北			橘子		100			null
东北			苹果		50			null
东北			葡萄		50			null
东北			柠檬		30			null
关东			柠檬		100			null
关东			苹果		100			null
关东			菠萝		100			null
关东			葡萄		70			null
关西			柠檬		70			null
关西			西瓜		30			null
关西			苹果		20			null

------------
----目标：---
------------

district	 name	   price	  ranking
东北			橘子		100			1
东北			苹果		50			2
东北			葡萄		50			2
东北			柠檬		30			4
关东			柠檬		100			1
关东			苹果		100			1
关东			菠萝		100			1
关东			葡萄		70			4
关西			柠檬		70			1
关西			西瓜		30			2
关西			苹果		20			3
```

```sql
-- MySQL5.7以上不能这么写
update district_products2 t1 set t1.ranking = (
	select count(t2.name)+1 from district_products2 t2 where t1.district=t2.district and t1.price<t2.price
);


-- MySQL5.7版本正确写法(不唯一)
-- 通过自连接或者是子查询查出ranking信息作为一个临时表，更新的时候将源表与临时表进行关联更新
update district_products2 t1 set t1.ranking = (
	select ranking from (
		select 
			district, name, price,
			(select count(p2.name)+1 from district_products2 p2 where p1.district=p2.district and p1.price<p2.price) as ranking 
		from district_products2 p1
	) t2 where t1.district=t2.district and t1.`name`=t2.`name`
);

```

## 3、HAVING子句

> HAVING子句可以脱离GROUP BY语句单独使用，但是单独使用了HAVING的话在select子句中就不能引用表中的列了，只能用常量或者使用聚合函数

#### 例题

#### 1、查询数据是否缺失

```sql
select * from seqtbl;

seq	name
1	迪克
2	安
3	莱露
5	卡
6	玛丽
8	本

-- 判断数据量与seq最大值是否相等
select '缺失' as flag from seqtbl having count(1) <> max(seq);
```

#### 2. 查询中位数

```sql
name	income				   
劳伦斯	15000
史密斯	20000
哈德逊	15000
怀特	20000
斯科特	10000
桑普森	400000
肯特	10000
贝克	10000
迈克	30000
阿诺德	20000


SELECT AVG(DISTINCT income)
  FROM (SELECT T1.income
          FROM Graduates T1, Graduates T2
      GROUP BY T1.income
               /* S1的条件 */
        HAVING SUM(CASE WHEN T2.income >= T1.income THEN 1 ELSE 0 END) 
                   >= COUNT(*) / 2
               /* S2的条件 */
           AND SUM(CASE WHEN T2.income <= T1.income THEN 1 ELSE 0 END) 
                   >= COUNT(*) / 2 ) TMP;
```

#### 3. count的使用

> count(*)可以用于null值，而count(列名)则会过滤掉列值为null的列