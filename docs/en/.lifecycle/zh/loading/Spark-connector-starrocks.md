---
displayed_sidebar: English
---


```markdown
- 从1.1.1版本开始，由于`mysql-connector-java`使用的GPL许可证的限制，Spark连接器不再提供MySQL的官方JDBC驱动程序。但是，Spark连接器仍然需要MySQL JDBC驱动程序来连接到StarRocks以获取表元数据，因此您需要手动将驱动程序添加到Spark类路径中。您可以在[MySQL官网](https://dev.mysql.com/downloads/connector/j/)或[Maven Central](https://repo1.maven.org/maven2/mysql/mysql-connector-java/)上找到该驱动程序。
- 从1.1.1版本开始，默认使用Stream Load接口而不是1.1.0版本中的Stream Load事务接口。如果您仍然希望使用Stream Load事务接口，可以将`starrocks.write.max.retries`选项设置为`0`。有关`starrocks.write.enable.transaction-stream-load`和`starrocks.write.max.retries`的详细信息，请参见相关描述。

## 示例

以下示例展示了如何使用Spark连接器通过Spark DataFrames或Spark SQL将数据加载到StarRocks表中。Spark DataFrames支持批处理和结构化流模式。

更多示例，请参见[Spark连接器示例](https://github.com/StarRocks/starrocks-connector-for-apache-spark/tree/main/src/test/java/com/starrocks/connector/spark/examples)。

### 准备工作

#### 创建StarRocks表

创建名为`test`的数据库，并创建一个主键表`score_board`。

```sql
CREATE DATABASE `test`;

CREATE TABLE `test`.`score_board`
(
    `id` int(11) NOT NULL COMMENT "",
    `name` varchar(65533) NULL DEFAULT "" COMMENT "",
    `score` int(11) NOT NULL DEFAULT "0" COMMENT ""
)
ENGINE=OLAP
PRIMARY KEY(`id`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`id`);
```

#### 设置Spark环境

请注意，以下示例在Spark 3.2.4中运行，并使用`spark-shell`、`pyspark`和`spark-sql`。在运行示例之前，请确保将Spark连接器JAR文件放置在`$SPARK_HOME/jars`目录中。

### 使用Spark DataFrames加载数据

以下两个示例说明了如何使用Spark DataFrames的批处理或结构化流模式加载数据。

#### 批处理

在内存中构造数据并将数据加载到StarRocks表中。

1. 您可以使用Scala或Python编写Spark应用程序。

对于Scala，在`spark-shell`中运行以下代码片段：

```scala
// 1. 从序列创建DataFrame。
val data = Seq((1, "starrocks", 100), (2, "spark", 100))
val df = data.toDF("id", "name", "score")

// 2. 通过配置格式为"starrocks"和以下选项将数据写入StarRocks。
// 您需要根据自己的环境修改选项。
df.write.format("starrocks")
    .option("starrocks.fe.http.url", "127.0.0.1:8030")
    .option("starrocks.fe.jdbc.url", "jdbc:mysql://127.0.0.1:9030")
    .option("starrocks.table.identifier", "test.score_board")
    .option("starrocks.user", "root")
    .option("starrocks.password", "")
    .mode("append")
    .save()
```

对于Python，在`pyspark`中运行以下代码片段：

```python
from pyspark.sql import SparkSession

spark = SparkSession \
     .builder \
     .appName("StarRocks示例") \
     .getOrCreate()

# 1. 从序列创建DataFrame。
data = [(1, "starrocks", 100), (2, "spark", 100)]
df = spark.sparkContext.parallelize(data) \
         .toDF(["id", "name", "score"])

# 2. 通过配置格式为"starrocks"和以下选项将数据写入StarRocks。
# 您需要根据自己的环境修改选项。
df.write.format("starrocks") \
     .option("starrocks.fe.http.url", "127.0.0.1:8030") \
     .option("starrocks.fe.jdbc.url", "jdbc:mysql://127.0.0.1:9030") \
     .option("starrocks.table.identifier", "test.score_board") \
     .option("starrocks.user", "root") \
     .option("starrocks.password", "") \
     .mode("append") \
     .save()
```

2. 在StarRocks表中查询数据。

```sql
MySQL [test]> SELECT * FROM `score_board`;
+------+-----------+-------+
| id   | name      | score |
+------+-----------+-------+
|    1 | starrocks |   100 |
|    2 | spark     |   100 |
+------+-----------+-------+
2行在集合中 (0.00秒)
```

#### 结构化流

从CSV文件中构造数据流读取，并将数据加载到StarRocks表中。

1. 在`csv-data`目录中，创建一个包含以下数据的CSV文件`test.csv`：

```csv
3,starrocks,100
4,spark,100
```

2. 您可以使用Scala或Python编写Spark应用程序。

对于Scala，在`spark-shell`中运行以下代码片段：

```scala
import org.apache.spark.sql.types.StructType

// 1. 从CSV创建DataFrame。
val schema = (new StructType()
        .add("id", "integer")
        .add("name", "string")
        .add("score", "integer")
    )
val df = (spark.readStream
        .option("sep", ",")
        .schema(schema)
        .format("csv")
        // 替换为您的"csv-data"目录路径。
        .load("/path/to/csv-data")
    )

// 2. 通过配置格式为"starrocks"和以下选项将数据写入StarRocks。
// 您需要根据自己的环境修改选项。
val query = (df.writeStream.format("starrocks")
        .option("starrocks.fe.http.url", "127.0.0.1:8030")
        .option("starrocks.fe.jdbc.url", "jdbc:mysql://127.0.0.1:9030")
        .option("starrocks.table.identifier", "test.score_board")
        .option("starrocks.user", "root")
        .option("starrocks.password", "")
        // 替换为您的检查点目录
        .option("checkpointLocation", "/path/to/checkpoint")
        .outputMode("append")
        .start()
    )
```

对于Python，在`pyspark`中运行以下代码片段：

```python
from pyspark.sql import SparkSession
from pyspark.sql.types import IntegerType, StringType, StructType, StructField

spark = SparkSession \
     .builder \
     .appName("StarRocks结构化流示例") \
     .getOrCreate()

# 1. 从CSV创建DataFrame。
schema = StructType([
        StructField("id", IntegerType()),
        StructField("name", StringType()),
        StructField("score", IntegerType())
    ])
df = (
    spark.readStream
    .option("sep", ",")
    .schema(schema)
    .format("csv")
    # 替换为您的"csv-data"目录路径。
    .load("/path/to/csv-data")
)

# 2. 通过配置格式为"starrocks"和以下选项将数据写入StarRocks。
# 您需要根据自己的环境修改选项。
query = (
    df.writeStream.format("starrocks")
    .option("starrocks.fe.http.url", "127.0.0.1:8030")
    .option("starrocks.fe.jdbc.url", "jdbc:mysql://127.0.0.1:9030")
    .option("starrocks.table.identifier", "test.score_board")
    .option("starrocks.user", "root")
    .option("starrocks.password", "")
    # 替换为您的检查点目录
    .option("checkpointLocation", "/path/to/checkpoint")
    .outputMode("append")
    .start()
)
```

3. 在StarRocks表中查询数据。

```sql
MySQL [test]> SELECT * FROM score_board;
+------+-----------+-------+
| id   | name      | score |
+------+-----------+-------+
|    4 | spark     |   100 |
|    3 | starrocks |   100 |
+------+-----------+-------+
2行在集合中 (0.67秒)
```

### 使用Spark SQL加载数据

以下示例说明了如何使用Spark SQL通过[Spark SQL CLI](https://spark.apache.org/docs/latest/sql-distributed-sql-engine-spark-sql-cli.html)中的`INSERT INTO`语句加载数据。

1. 在`spark-sql`中执行以下SQL语句：

```sql
-- 1. 通过配置数据源为`starrocks`和以下选项来创建表。
-- 您需要根据自己的环境修改选项。
CREATE TABLE `score_board`
USING starrocks
OPTIONS(
"starrocks.fe.http.url"="127.0.0.1:8030",
"starrocks.fe.jdbc.url"="jdbc:mysql://127.0.0.1:9030",
"starrocks.table.identifier"="test.score_board",
"starrocks.user"="root",
"starrocks.password"=""
);

-- 2. 向表中插入两行数据。
INSERT INTO `score_board` VALUES (5, "starrocks", 100), (6, "spark", 100);
```

2. 在StarRocks表中查询数据。

```sql
MySQL [test]> SELECT * FROM score_board;
+------+-----------+-------+
| id   | name      | score |
+------+-----------+-------+
|    6 | spark     |   100 |
|    5 | starrocks |   100 |
+------+-----------+-------+
2行在集合中 (0.00秒)
```

## 最佳实践

### 将数据加载到主键表

本节将展示如何将数据加载到StarRocks主键表以实现部分更新和条件更新。
您可以查看[通过加载更改数据](../loading/Load_to_Primary_Key_tables.md)以获取这些功能的详细介绍。
这些示例使用Spark SQL。

#### 准备工作

在StarRocks中创建名为`test`的数据库，并创建一个主键表`score_board`。

```sql
CREATE DATABASE `test`;

CREATE TABLE `test`.`score_board`
(
 `id` int(11) NOT NULL COMMENT "",
 `name` varchar(65533) NULL DEFAULT "" COMMENT "",
 `score` int(11) NOT NULL DEFAULT "0" COMMENT ""
)
ENGINE=OLAP
PRIMARY KEY(`id`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`id`);
```

#### 部分更新

此示例将展示如何通过加载仅更新`name`列中的数据：

1. 将初始数据插入MySQL客户端中的StarRocks表。

```sql
mysql> INSERT INTO `score_board` VALUES (1, 'starrocks', 100), (2, 'spark', 100);

mysql> SELECT * FROM score_board;
+------+-----------+-------+
| id   | name      | score |
+------+-----------+-------+
|    1 | starrocks |   100 |
|    2 | spark     |   100 |
+------+-----------+-------+
2行在集合中 (0.02秒)
```

2. 在Spark SQL客户端中创建一个名为`score_board`的Spark表。

- 将选项`starrocks.write.properties.partial_update`设置为`true`，这会告诉连接器执行部分更新。
- 将选项`starrocks.columns`设置为`"id,name"`，以告诉连接器要写入哪些列。

```sql
CREATE TABLE `score_board`
USING starrocks
OPTIONS(
   "starrocks.fe.http.url"="127.0.0.1:8030",
   "starrocks.fe.jdbc.url"="jdbc:mysql://127.0.0.1:9030",
   "starrocks.table.identifier"="test.score_board",
   "starrocks.user"="root",
   "starrocks.password"="",
   "starrocks.write.properties.partial_update"="true",
   "starrocks.columns"="id,name"
);
```

3. 在Spark SQL客户端中向表中插入数据，仅更新`name`列。

```sql
INSERT INTO `score_board` VALUES (1, 'starrocks-update'), (2, 'spark-update');
```

4. 在MySQL客户端中查询StarRocks表。

您可以看到，只有`name`的值发生了变化，而`score`的值没有变化。

```sql
mysql> SELECT * FROM score_board;
+------+------------------+-------+
| id   | name             | score |
+------+------------------+-------+
|    1 | starrocks-update |   100 |
|    2 | spark-update     |   100 |
+------+------------------+-------+
2行在集合中 (0.02秒)
```
```
```SQL
INSERT INTO `score_board` VALUES (1, 'starrocks-update'), (2, 'spark-update');
```

4. 在 MySQL 客户端中查询 StarRocks 表。

   您可以看到，只有 `name` 的值发生了变化，`score` 的值没有变化。

   ```SQL
   mysql> SELECT * FROM `score_board`;
   +------+------------------+-------+
   | id   | name             | score |
   +------+------------------+-------+
   |    1 | starrocks-update |   100 |
   |    2 | spark-update     |   100 |
   +------+------------------+-------+
   2 rows in set (0.02 sec)
   ```

#### 条件更新

此示例将展示如何根据列 `score` 的值进行条件更新。仅当新的 `score` 值大于或等于旧值时，对应 `id` 的更新才会生效。

1. 将初始数据插入 MySQL 客户端中的 StarRocks 表。

   ```SQL
   mysql> INSERT INTO `score_board` VALUES (1, 'starrocks', 100), (2, 'spark', 100);
   
   mysql> SELECT * FROM `score_board`;
   +------+-----------+-------+
   | id   | name      | score |
   +------+-----------+-------+
   |    1 | starrocks |   100 |
   |    2 | spark     |   100 |
   +------+-----------+-------+
   2 rows in set (0.02 sec)
   ```

2. 通过以下方式创建 Spark 表 `score_board`。

   - 将选项 `starrocks.write.properties.merge_condition` 设置为 `score`，告诉连接器使用列 `score` 作为条件。
   - 确保 Spark 连接器使用 Stream Load 接口加载数据，而不是 Stream Load 事务接口，因为后者不支持此功能。

   ```SQL
   CREATE TABLE `score_board`
   USING starrocks
   OPTIONS(
       "starrocks.fe.http.url"="127.0.0.1:8030",
       "starrocks.fe.jdbc.url"="jdbc:mysql://127.0.0.1:9030",
       "starrocks.table.identifier"="test.score_board",
       "starrocks.user"="root",
       "starrocks.password"="",
       "starrocks.write.properties.merge_condition"="score"
    );
   ```

3. 在 Spark SQL 客户端向表中插入数据，将 `id` 为 1 的行更新为 `score` 值较小的行，将 `id` 为 2 的行更新为 `score` 值较大的行。

   ```SQL
   INSERT INTO `score_board` VALUES (1, 'starrocks-update', 99), (2, 'spark-update', 101);
   ```

4. 在 MySQL 客户端中查询 StarRocks 表。

   可以看到只有 `id` 为 2 的行发生了变化，`id` 为 1 的行没有发生变化。

   ```SQL
   mysql> SELECT * FROM `score_board`;
   +------+--------------+-------+
   | id   | name         | score |
   +------+--------------+-------+
   |    1 | starrocks    |   100 |
   |    2 | spark-update |   101 |
   +------+--------------+-------+
   2 rows in set (0.03 sec)
   ```

### 将数据加载到 BITMAP 类型的列中

[`BITMAP`](../sql-reference/sql-statements/data-types/BITMAP.md) 常用于加速 Count Distinct，例如统计 UV，请参见 [使用 Bitmap 进行精确 Count Distinct](../using_starrocks/Using_bitmap.md)。
这里以 UV 计数为例，展示如何将数据加载到 `BITMAP` 类型的列中。**`BITMAP` 是从 1.1.1 版本开始支持的**。

1. 创建 StarRocks 聚合表。

   在数据库 `test` 中，创建聚合表 `page_uv`，其中列 `visit_users` 定义为 `BITMAP` 类型，并配置聚合函数 `BITMAP_UNION`。

   ```SQL
   CREATE TABLE `test`.`page_uv` (
     `page_id` INT NOT NULL COMMENT '页面 ID',
     `visit_date` DATETIME NOT NULL COMMENT '访问时间',
     `visit_users` BITMAP BITMAP_UNION NOT NULL COMMENT '用户 ID'
   ) ENGINE=OLAP
   AGGREGATE KEY(`page_id`, `visit_date`)
   DISTRIBUTED BY HASH(`page_id`);
   ```

2. 创建 Spark 表。

   Spark 表的 schema 是从 StarRocks 表推断出来的，Spark 不支持 `BITMAP` 类型。因此，您需要在 Spark 中自定义相应的列数据类型，例如 `BIGINT`，通过配置选项 `"starrocks.column.types"="visit_users BIGINT"`。当使用 Stream Load 摄取数据时，连接器使用 [`to_bitmap`](../sql-reference/sql-functions/bitmap-functions/to_bitmap.md) 函数将 `BIGINT` 类型的数据转换为 `BITMAP` 类型。

   在 spark-sql 中运行以下 DDL：

   ```SQL
   CREATE TABLE `page_uv`
   USING starrocks
   OPTIONS(
      "starrocks.fe.http.url"="127.0.0.1:8030",
      "starrocks.fe.jdbc.url"="jdbc:mysql://127.0.0.1:9030",
      "starrocks.table.identifier"="test.page_uv",
      "starrocks.user"="root",
      "starrocks.password"="",
      "starrocks.column.types"="visit_users BIGINT"
   );
   ```

3. 将数据加载到 StarRocks 表中。

   在 spark-sql 中运行以下 DML：

   ```SQL
   INSERT INTO `page_uv` VALUES
      (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 13),
      (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 23),
      (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 33),
      (1, CAST('2020-06-23 02:30:30' AS TIMESTAMP), 13),
      (2, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 23);
   ```

4. 从 StarRocks 表计算页面 UV。

   ```SQL
   MySQL [test]> SELECT `page_id`, COUNT(DISTINCT `visit_users`) FROM `page_uv` GROUP BY `page_id`;
   +---------+-----------------------------+
   | page_id | COUNT(DISTINCT `visit_users`) |
   +---------+-----------------------------+
   |       2 |                           1 |
   |       1 |                           3 |
   +---------+-----------------------------+
   2 rows in set (0.01 sec)
   ```

> **注意：**
> 连接器使用 [`to_bitmap`](../sql-reference/sql-functions/bitmap-functions/to_bitmap.md) 函数将 Spark 中的 `TINYINT`、`SMALLINT`、`INTEGER` 和 `BIGINT` 类型的数据转换为 StarRocks 中的 `BITMAP` 类型，并使用 [`bitmap_hash`](../sql-reference/sql-functions/bitmap-functions/bitmap_hash.md) 函数将 Spark 中的其他数据类型转换为 `BITMAP` 类型。

### 将数据加载到 HLL 类型的列中

[`HLL`](../sql-reference/sql-statements/data-types/HLL.md) 可用于近似非重复计数，请参阅 [使用 HLL 进行近似非重复计数](../using_starrocks/Using_HLL.md)。

这里我们以 UV 的计数为例，展示如何将数据加载到 `HLL` 类型的列中。**`HLL` 是从 1.1.1 版本开始支持的**。

1. 创建 StarRocks 聚合表。

   在数据库 `test` 中，创建聚合表 `hll_uv`，其中列 `visit_users` 定义为 `HLL` 类型，并配置聚合函数 `HLL_UNION`。

   ```SQL
   CREATE TABLE `hll_uv` (
   `page_id` INT NOT NULL COMMENT '页面 ID',
   `visit_date` DATETIME NOT NULL COMMENT '访问时间',
   `visit_users` HLL HLL_UNION NOT NULL COMMENT '用户 ID'
   ) ENGINE=OLAP
   AGGREGATE KEY(`page_id`, `visit_date`)
   DISTRIBUTED BY HASH(`page_id`);
   ```

2. 创建 Spark 表。

   Spark 表的 schema 是从 StarRocks 表推断出来的，Spark 不支持 `HLL` 类型。因此，您需要在 Spark 中自定义相应的列数据类型，例如 `BIGINT`，通过配置选项 `"starrocks.column.types"="visit_users BIGINT"`。使用 Stream Load 摄取数据时，连接器使用 [`hll_hash`](../sql-reference/sql-functions/aggregate-functions/hll_hash.md) 函数将 `BIGINT` 类型的数据转换为 `HLL` 类型。

   在 spark-sql 中运行以下 DDL：

   ```SQL
   CREATE TABLE `hll_uv`
   USING starrocks
   OPTIONS(
      "starrocks.fe.http.url"="127.0.0.1:8030",
      "starrocks.fe.jdbc.url"="jdbc:mysql://127.0.0.1:9030",
      "starrocks.table.identifier"="test.hll_uv",
      "starrocks.user"="root",
      "starrocks.password"="",
      "starrocks.column.types"="visit_users BIGINT"
   );
   ```

3. 将数据加载到 StarRocks 表中。

   在 spark-sql 中运行以下 DML：

   ```SQL
   INSERT INTO `hll_uv` VALUES
      (3, CAST('2023-07-24 12:00:00' AS TIMESTAMP), 78),
      (4, CAST('2023-07-24 13:20:10' AS TIMESTAMP), 2),
      (3, CAST('2023-07-24 12:30:00' AS TIMESTAMP), 674);
   ```

4. 从 StarRocks 表计算页面 UV。

   ```SQL
   MySQL [test]> SELECT `page_id`, COUNT(DISTINCT `visit_users`) FROM `hll_uv` GROUP BY `page_id`;
   +---------+-----------------------------+
   | page_id | COUNT(DISTINCT `visit_users`) |
   +---------+-----------------------------+
   |       4 |                           1 |
   |       3 |                           2 |
   +---------+-----------------------------+
   2 rows in set (0.01 sec)
   ```

### 将数据加载到 ARRAY 类型的列中

以下示例说明如何将数据加载到 [`ARRAY`](../sql-reference/sql-statements/data-types/Array.md) 类型的列中。

1. 创建一个 StarRocks 表。

   在数据库 `test` 中，创建一个主键表 `array_tbl`，其中包含 1 个 `INT` 列和 2 个 `ARRAY` 列。

   ```SQL
   CREATE TABLE `array_tbl` (
       `id` INT NOT NULL,
       `a0` ARRAY<STRING>,
       `a1` ARRAY<ARRAY<INT>>
    )
    ENGINE=OLAP
    PRIMARY KEY(`id`)
    DISTRIBUTED BY HASH(`id`)
    ;
   ```

2. 将数据写入 StarRocks。

   由于某些版本的 StarRocks 不提供 `ARRAY` 列的元数据，连接器无法推断该列对应的 Spark 数据类型。但是，您可以在选项 `starrocks.column.types` 中显式指定列的 Spark 数据类型。在此示例中，您可以将该选项配置为 `a0 ARRAY<STRING>,a1 ARRAY<ARRAY<INT>>`。

   在 spark-shell 中运行以下代码：

   ```scala
   val data = Seq(
       (1, Seq("hello", "starrocks"), Seq(Seq(1, 2), Seq(3, 4))),
       (2, Seq("hello", "spark"), Seq(Seq(5, 6, 7), Seq(8, 9, 10)))
   )
   val df = data.toDF("id", "a0", "a1")
   df.write
       .format("starrocks")
       .option("starrocks.fe.http.url", "127.0.0.1:8030")
       .option("starrocks.fe.jdbc.url", "jdbc:mysql://127.0.0.1:9030")
       .option("starrocks.table.identifier", "test.array_tbl")
       .option("starrocks.user", "root")
       .option("starrocks.password", "")
       .option("starrocks.column.types", "a0 ARRAY<STRING>,a1 ARRAY<ARRAY<INT>>")
       .mode("append")
       .save()
   ```

3. 在 StarRocks 表中查询数据。

   ```SQL
   MySQL [test]> SELECT * FROM `array_tbl`;
   +------+-----------------------+--------------------+
   | id   | a0                    | a1                 |
   +------+-----------------------+--------------------+
   |    1 | ["hello","starrocks"] | [[1,2],[3,4]]      |
   |    2 | ["hello","spark"]     | [[5,6,7],[8,9,10]] |
   +------+-----------------------+--------------------+
   2 rows in set (0.01 sec)
   ```
```markdown
         .mode("append")
         .save()
   ```

3. 查询StarRocks表中的数据。

   ```SQL
   MySQL [test]> SELECT * FROM `array_tbl`;
   +------+-----------------------+--------------------+
   | id   | a0                    | a1                 |
   +------+-----------------------+--------------------+
   |    1 | ["hello","starrocks"] | [[1,2],[3,4]]      |
   |    2 | ["hello","spark"]     | [[5,6,7],[8,9,10]] |
   +------+-----------------------+--------------------+
   2行在集合中 (0.01秒)
   ```
```