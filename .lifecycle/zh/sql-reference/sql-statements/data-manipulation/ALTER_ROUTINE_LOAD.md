---
displayed_sidebar: English
---

# 修改例行加载作业

## 描述

此命令用于修改已创建的例行加载（Routine Load）作业。要修改的作业必须处于**暂停**（PAUSED）状态。您可以运行[PASUME](./PAUSE_ROUTINE_LOAD.md)(PAUSE ROUTINE LOAD)命令以暂停加载作业，然后对该作业执行`ALTER ROUTINE LOAD`操作。

## 语法

> 注意：您无需指定方括号[]内的内容。

```SQL
ALTER ROUTINE LOAD FOR [db.]<job_name>

[load_properties]

[job_properties]

FROM data_source

[data_source_properties]
```

## **参数**

- **`[db.]	extless job_name	extgreater`**

您想要修改的作业名称。

- **load_properties**

要导入的数据属性。

语法：

```SQL
[COLUMNS TERMINATED BY '<terminator>'],

[COLUMNS ([<column_name> [, ...] ] [, column_assignment [, ...] ] )],

[WHERE <expr>],

[PARTITION ([ <partition_name> [, ...] ])]



column_assignment:

<column_name> = column_expression
```

1. 指定列分隔符。

   您可以为想要导入的CSV数据指定列分隔符。例如，您可以使用逗号（,）作为列分隔符。

   ```SQL
   COLUMNS TERMINATED BY ","
   ```

   默认分隔符是制表符（\t）。

2. 指定列映射。

   指定源表与目标表中列的映射，并定义派生列的生成方式。

   - 映射列

   指定源表中哪些列按序对应目标表中的哪些列。如果您想要跳过某一列，可以指定一个不存在的列名。例如，目标表有三列k1、k2和v1，源表有四列，其中第一、二和第四列分别对应k2、k1和v1。您可以如下编写代码：

   ```SQL
   COLUMNS (k2, k1, xxx, v1)
   ```

   xxx是一个不存在的列名，用来跳过源表中的第三列。

   - 派生列

   以col_name = expr的形式表示的列为派生列。这些列通过表达式expr生成。派生列通常放在映射列之后。虽然这不是一个强制规则，但StarRocks总是先解析映射列再解析派生列。假设目标表有一个第四列v2，通过累加k1和k2生成。您可以如下编写代码：

   ```plaintext
   COLUMNS (k2, k1, xxx, v1, v2 = k1 + k2);
   ```

   对于CSV数据，COLUMNS中映射的列数必须与CSV文件中的列数一致。

3. 指定过滤条件。

   您可以设置过滤条件以过滤掉不需要的列。过滤列可以是映射列或派生列。例如，如果您只想导入k1值大于100且k2值等于1000的列数据，可以如下编写代码：

   ```plaintext
   WHERE k1 > 100 and k2 = 1000
   ```

4. 指定要导入数据的分区。

   如果未指定分区，数据将根据CSV数据中的分区键值自动导入到StarRocks的分区中。例如：

   ```plaintext
   PARTITION(p1, p2, p3)
   ```

- **job_properties**

您想要修改的作业参数。目前，您可以修改以下参数：

1. desired_concurrent_number（期望的并发数）
2. max_error_number（最大错误数）
3. max_batch_interval（最大批次间隔）
4. max_batch_rows（最大批次行数）
5. max_batch_size（最大批量大小）
6. jsonpaths
7. json_root
8. strip_outer_array
9. strict_mode（严格模式）
10. timezone（时区）

- **data_source**

数据源的类型。目前只支持Kafka数据源。

- **data_source_properties**

数据源的属性。支持以下属性：

1. kafka_partitions

   您只能修改已经消费的Kafka分区。

2. kafka_offsets

   您只能修改尚未消费的分区偏移量。

3. 自定义属性，如property.group.id和property.group.id

> 您只能在kafka_partitions中指定已消费的Kafka分区。在kafka_offsets中，您只能指定尚未消费的分区偏移量。

## 示例

示例1：将desired_concurrent_number的值更改为1，此参数指定用于消费Kafka数据的作业并行度。

```SQL
ALTER ROUTINE LOAD FOR db1.label1

PROPERTIES

(

    "desired_concurrent_number" = "1"

);
```

示例2：将desired_concurrent_number的值更改为10，并修改分区偏移和组ID。

```SQL
ALTER ROUTINE LOAD FOR db1.label1

PROPERTIES

(

    "desired_concurrent_number" = "10"

)

FROM kafka

(

    "kafka_partitions" = "0, 1, 2",

    "kafka_offsets" = "100, 200, 100",

    "property.group.id" = "new_group"

);
```

示例3：将过滤条件更改为a > 0，并将目标分区设置为p1。

```SQL
ALTER ROUTINE LOAD FOR db1.label1

WHERE a > 0

PARTITION (p1)
```
