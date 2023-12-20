---
displayed_sidebar: English
---

# 修改 ROUTINE LOAD 任务

## 描述

该命令用于修改已创建的 **Routine Load** 作业。要修改的作业必须处于 **PAUSED** 状态。您可以运行 [PAUSE](./PAUSE_ROUTINE_LOAD.md)（暂停 ROUTINE LOAD）命令来暂停一个加载作业，然后对该作业执行 `ALTER ROUTINE LOAD` 操作。

## 语法

> 注意：您不需要指定方括号 [] 中的内容。

```SQL
ALTER ROUTINE LOAD FOR [db.]<job_name>

[load_properties]

[job_properties]

FROM data_source

[data_source_properties]
```

## **参数**

- **`[db.]<job_name>`**

您要修改的作业名称。

- **load_properties**

要导入数据的属性。

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

   您可以为要导入的 CSV 数据指定列分隔符。例如，您可以使用逗号 (,) 作为列分隔符。

   ```SQL
   COLUMNS TERMINATED BY ","
   ```

   默认分隔符是 `\t`。

2. 指定列映射。

   指定源表与目标表中的列映射，并定义派生列的生成方式。

   - 映射列

   指定源表中的哪些列按顺序对应目标表中的哪些列。如果您想跳过某列，可以指定一个不存在的列名。例如，目标表有三列 k1、k2 和 v1，源表有四列，其中第一、二和第四列分别对应 k2、k1 和 v1。您可以这样编写代码：

   ```SQL
   COLUMNS (k2, k1, xxx, v1)
   ```

   `xxx` 是一个不存在的列，用来跳过源表中的第三列。

   - 派生列

   以 `col_name = expr` 形式表示的列是派生列。这些列通过使用 `expr` 生成。派生列通常放在映射列之后。虽然这不是一个强制规则，但 StarRocks 总是先解析映射列，再解析派生列。假设目标表有第四列 v2，它是通过 k1 和 k2 相加得到的。您可以这样编写代码：

   ```plaintext
   COLUMNS (k2, k1, xxx, v1, v2 = k1 + k2);
   ```

   对于 CSV 数据，COLUMNS 中映射的列数必须与 CSV 文件中的列数一致。

3. 指定过滤条件。

   您可以指定过滤条件来筛选出不需要的列。过滤条件可以应用于映射列或派生列。例如，如果您只想导入 k1 值大于 100 且 k2 值等于 1000 的列，您可以这样编写代码：

   ```plaintext
   WHERE k1 > 100 AND k2 = 1000
   ```

4. 指定要导入数据的分区。

   如果没有指定分区，数据将根据 CSV 数据中的分区键值自动导入到 StarRocks 的分区中。例如：

   ```plaintext
   PARTITION(p1, p2, p3)
   ```

- **job_properties**

您想要修改的作业参数。目前，您可以修改以下参数：

1. `desired_concurrent_number`
2. `max_error_number`
3. `max_batch_interval`
4. `max_batch_rows`
5. `max_batch_size`
6. `jsonpaths`
7. `json_root`
8. `strip_outer_array`
9. `strict_mode`
10. `timezone`

- **data_source**

数据源的类型。目前只支持 Kafka 数据源。

- **data_source_properties**

数据源的属性。支持以下属性：

1. `kafka_partitions`

   您只能修改已经消费的 Kafka 分区。

2. `kafka_offsets`

   您只能修改尚未消费的分区偏移。

3. 自定义属性，如 `property.group.id` 和 `property.group.id`

> 您只能在 `kafka_partitions` 中指定已经消费的 Kafka 分区。您只能在 `kafka_offsets` 中指定尚未消费的分区偏移。

## 示例

示例 1：将 `desired_concurrent_number` 的值更改为 1。该参数指定用于消费 Kafka 数据的作业的并行度。

```SQL
ALTER ROUTINE LOAD FOR db1.label1

PROPERTIES

(

    "desired_concurrent_number" = "1"

);
```

示例 2：将 `desired_concurrent_number` 的值更改为 10，并修改分区偏移和组 ID。

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

示例 3：将过滤条件更改为 `a > 0` 并设置目标分区为 `p1`。

```SQL
ALTER ROUTINE LOAD FOR db1.label1

WHERE a > 0

PARTITION (p1)
```