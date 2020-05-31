## clickhouse SQL使用参考

```sql
[WITH expr_list|(subquery)]
SELECT [DISTINCT] expr_list
[FROM [db.]table | (subquery) | table_function] [FINAL]
[SAMPLE sample_coeff]
[ARRAY JOIN ...]
[GLOBAL] [ANY|ALL] [INNER|LEFT|RIGHT|FULL|CROSS] [OUTER] JOIN (subquery)|table USING columns_list
[PREWHERE expr]
[WHERE expr]
[GROUP BY expr_list] [WITH TOTALS]
[HAVING expr]
[ORDER BY expr_list]
[LIMIT [offset_value, ]n BY columns]
[LIMIT [n, ]m]
[UNION ALL ...]
[INTO OUTFILE filename]
[FORMAT format]
```

所有子句都是可选的，但SELECT之后的映射区是必需的。下面的子句以与查询执行传送器几乎相同的顺序描述。

### with

> 限制条件：
>
> 1.不支持递归查询
>
> 2.当在with部分中使用子查询时，其结果应恰好只有一行
>
> 3.表达式的结果在子查询中不可用

#### 1. 将常量表达式用作变量

```sql
WITH '2019-08-01 15:23:00' as ts_upper_bound
SELECT *
FROM hits
WHERE
    EventDate = toDate(ts_upper_bound) AND
    EventTime <= ts_upper_bound
```

#### 2. 对映射区的列进行计算

```sql
WITH sum(bytes) as s
SELECT
    formatReadableSize(s) as size,
    table
FROM system.parts
GROUP BY table
ORDER BY s

┌─size───────┬─table──────┐
│ 744.00 B   │ mt_table   │
│ 11.90 KiB  │ trace_log  │
│ 149.04 MiB │ metric_log │
│ 536.00 MiB │ visits_v1  │
│ 1.20 GiB   │ hits_v1    │
└────────────┴────────────┘
```

#### 3. 使用标量子查询的结果

```sql
/* this example would return TOP 10 of most huge tables */
WITH
    (
        SELECT sum(bytes)
        FROM system.parts
        WHERE active
    ) AS total_disk_usage
SELECT
    (sum(bytes) / total_disk_usage) * 100 AS table_disk_usage,
    table
FROM system.parts
GROUP BY table
ORDER BY table_disk_usage DESC
LIMIT 10

┌───────table_disk_usage─┬─table──────┐
│      69.20777669996204 │ hits_v1    │
│      30.19938472709433 │ visits_v1  │
│      8.428774092841918 │ metric_log │
│  0.0006549895597216437 │ trace_log  │
│ 0.00003997639314461878 │ mt_table   │
└────────────────────────┴────────────┘
```

#### 4. 在子查询中重用表达式

```sql
WITH ['hello'] AS hello
SELECT
    hello,
    *
FROM
(
    WITH ['hello'] AS hello
    SELECT hello
)

┌─hello─────┬─hello─────┐
│ ['hello'] │ ['hello'] │
└───────────┴───────────┘
```

### from

> FROM子句指定从中读取数据的源：
>
> - Table
> - Subquery
> - [Table function](https://clickhouse.tech/docs/en/sql_reference/table_functions/)

可以指定SELECT子查询来代替表。

与标准SQL相比，不需要在子查询之后指定同义词。

要执行查询，将从表中提取查询所有列。如果查询未列出列（例如SELECT count（）FROM t），则无论如何都要从表中提取一些列（最好是最小的列），以便计算行数。

### sample

> 启用数据采样后，不会对所有数据执行查询，而是仅对特定部分数据（采样）执行查询。例如，如果您需要计算所有访问的统计信息，那么只需对所有访问的1/10分数执行查询，然后将结果乘以10就足够了。

#### **适用的场景：**

- 如果您对查询时间有严格的要求（例如<100ms），但无法增加硬件资源。
- 如果原始数据不准确，则近似值不会明显降低质量。
- 业务需求的目标是大致的结果（出于成本效益，或为了向高级用户销售准确的结果）。

#### **note:**

```
仅在表创建期间指定了采样表达式的情况下，才可以对MergeTree家族中的表使用采样（请参见MergeTree引擎）。
```

#### **数据采样的特点：**

- 数据采样是确定性机制。相同的SELECT ..SAMPLE查询的结果始终相同。
- 采样对于不同的表一致地工作。对于具有单个采样列的表，如果是同一个采样系数，选择的数据集可能始终相同。例如，一个用户ID样本从不同的表中获取具有所有可能的用户ID的相同子集的行。这意味着可以在IN子句的子查询中使用样本。另外，您可以使用JOIN子句加入示例。
- 采样允许从磁盘读取更少的数据。但是必须正确指定采样的列。

#### **SAMPLE子句支持的语法：**

| 使用语句          | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| SAMPLE k          | 0<k<1，表示从总数据中取出k的比例进行查询，例如k=0.1表示取10%的数据 |
| SAMPLE n          | n为一个具体的数值，表示取多少行数据进行查询，例如n=100000,表示取10w行 |
| SAMPLE k OFFSET m | k和m都是一个0到1之间的数，k表示取的比例，m表示偏移量         |

- SAMPLE k

  ```sql
  SELECT
      Title,
      count() * 10 AS PageViews
  FROM hits_distributed
  SAMPLE 0.1
  WHERE
      CounterID = 34
  GROUP BY Title
  ORDER BY PageViews DESC LIMIT 1000
  ```

- SAMPLE n

  ```sql
  SELECT sum(PageViews * _sample_factor)
  FROM visits
  SAMPLE 10000000
  ```

- SAMPLE k OFFSET m

  ```sql
  SAMPLE 1/10 OFFSET 1/2
  
  取的数据的位置信息：
  [------++------]
  ```

### array join

> 允许使用数组或嵌套数据结构执行JOIN。目的类似于arrayJoin函数，但其功能更广泛

```sql
SELECT <expr_list>
FROM <left_subquery>
[LEFT] ARRAY JOIN <array>
[WHERE|PREWHERE <expr>]
...
```

```sql
CREATE TABLE arrays_test
(
    s String,
    arr Array(UInt8)
) ENGINE = Memory;

INSERT INTO arrays_test
VALUES ('Hello', [1,2]), ('World', [3,4,5]), ('Goodbye', []);

select * from arrays_test;

┌─s───────────┬─arr─────┐
│ Hello       │ [1,2]   │
│ World       │ [3,4,5] │
│ Goodbye     │ []      │
└─────────────┴─────────┘
```

#### 表达式类型

- array join：结果不包含空列

  ```sql
  SELECT s, arr
  FROM arrays_test
  ARRAY JOIN arr;
  
  ┌─s─────┬─arr─┐
  │ Hello │   1 │
  │ Hello │   2 │
  │ World │   3 │
  │ World │   4 │
  │ World │   5 │
  └───────┴─────┘
  ```

- left array join：结果包含空列

  ```sql
  SELECT s, arr
  FROM arrays_test
  LEFT ARRAY JOIN arr;
  
  ┌─s───────────┬─arr─┐
  │ Hello       │   1 │
  │ Hello       │   2 │
  │ World       │   3 │
  │ World       │   4 │
  │ World       │   5 │
  │ Goodbye     │   0 │
  └─────────────┴─────┘
  ```

  

