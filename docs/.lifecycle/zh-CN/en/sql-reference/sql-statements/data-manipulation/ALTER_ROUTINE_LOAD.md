---
displayed_sidebar: "Chinese"
---

# 修改常规加载

## 描述

此命令用于修改已创建的常规加载作业。要修改的作业必须处于**暂停**状态。您可以运行[PASUME](./PAUSE_ROUTINE_LOAD.md)(PAUSE ROUTINE LOAD)命令来暂停加载作业，然后对作业执行`ALTER ROUTINE LOAD`操作。

## 语法

> 注意：您不需要指定方括号[]中的内容。

```SQL
ALTER ROUTINE LOAD FOR [db.]<job_name>

[load_properties]

[job_properties]

FROM 数据源

[data_source_properties]
```

## **参数**

- **`[db.]<job_name>`**

要修改的作业的名称。

- **load_properties**

要导入的数据的属性。

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

   您可以为要导入的CSV数据指定列分隔符。例如，您可以使用逗号(,)作为列分隔符。

    ```SQL
    COLUMNS TERMINATED BY ","
    ```

   默认分隔符是`\t`。

2. 指定列映射。

   指定源表和目标表中列的映射，并定义如何生成派生列。

   - 映射列

   按顺序指定源表中的哪些列对应于目标表中的哪些列。如果要跳过一列，可以指定一个不存在的列名。例如，目标表有三列k1、k2和v1。源表有四列，其中第一、第二和第四列对应于k2、k1和v1。您可以编写如下代码。

    ```SQL
    COLUMNS (k2, k1, xxx, v1)
    ```

   `xxx`是不存在的列，用于跳过源表中的第三列。

   - 派生列

   表达为`col_name = expr`的列是派生列。这些列通过使用`expr`生成。派生列通常放置在映射列之后。虽然这不是强制规则，但StarRocks始终优先解析映射列而不是派生列。假设目标表有第四列v2，由k1和k2相加生成。您可以编写如下代码。

    ```plaintext
    COLUMNS (k2, k1, xxx, v1, v2 = k1 + k2);
    ```

   对于CSV数据，COLUMNS中的映射列数量必须与CSV文件中的列数匹配。

3. 指定过滤条件。

   您可以指定过滤条件来过滤不需要的列。过滤列可以是映射列或派生列。例如，如果需要从k1值大于100且k2值等于1000的列中导入数据，可以编写如下代码。

    ```plaintext
    WHERE k1 > 100 and k2 = 1000
    ```

4. 指定要导入数据的分区。

   如果未指定分区，将根据CSV数据中的分区键值自动将数据导入StarRocks分区。例如：

    ```plaintext
    PARTITION(p1, p2, p3)
    ```

- **job_properties**

要修改的作业参数。目前，您可以修改以下参数：

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

数据源的类型。目前仅支持Kafka数据源。

- **data_source_properties**

数据源的属性。支持以下属性：

1. `kafka_partitions`

   您只能修改已消费的Kafka分区。

2. `kafka_offsets`

   您只能修改尚未被消费的分区偏移量。

3. 诸如`property.group.id`和`property.group.id`之类的自定义属性

> 在`kafka_partitions`中只能指定已消费的Kafka分区。在`kafka_offsets`中只能指定尚未被消费的分区偏移量。

## 示例

示例1：将`desired_concurrent_number`的值更改为1。此参数指定用于消费Kafka数据的作业的并行度。

```SQL
ALTER ROUTINE LOAD FOR db1.label1

PROPERTIES

(

    "desired_concurrent_number" = "1"

);
```

示例2：将`desired_concurrent_number`的值更改为10，并修改分区偏移和组ID。

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

示例3：将过滤条件更改为`a > 0`，并设置目标分区为`p1`。

```SQL
ALTER ROUTINE LOAD FOR db1.label1

WHERE a > 0

PARTITION (p1)
```