---
displayed_sidebar: "Chinese"
---

# 列表分区

自 v3.1 版本起，StarRocks 支持列表分区。数据根据每个分区的预定义值列表进行分区，这可以加快查询速度，便于根据枚举值进行管理。

## 介绍

您需要明确指定每个分区中的列值列表。与范围分区中需要连续的时间或数字范围不同，这些值不需要连续。在数据加载过程中，StarRocks 将根据数据的分区列值与每个分区的预定义列值之间的映射将数据存储在相应的分区中。

![list_partitioning](../assets/list_partitioning.png)

列表分区适用于存储列包含少量枚举值的数据，并且经常根据这些枚举值查询和管理数据。例如，列代表地理位置、州和类别。列中的每个值代表一个独立的类别。通过基于枚举值对数据进行分区，您可以提高查询性能并便于数据管理。

**列表分区特别适用于分区需要包含每个分区列的多个值的情况**。例如，如果表中包括代表个人所在城市的 `City` 列，并且您经常根据州和城市查询和管理数据。在表创建时，您可以使用 `City` 列作为列表分区的分区列，并指定同一州内各城市的数据放在一个分区中，例如 `PARTITION pCalifornia VALUES IN ("Los Angeles", "San Francisco", "San Diego")`。这种方法可以加速根据州和城市进行查询，同时便于进行数据管理。

如果一个分区只需包含分区列的相同值的数据，建议使用[表达式分区](./expression_partitioning.md)。

**列表分区与[表达式分区](./expression_partitioning.md)的比较**

列表分区和表达式分区（建议）的主要区别在于，列表分区需要您手动创建分区。另一方面，表达式分区可以在加载数据时自动创建分区，简化分区。并且在大多数情况下，表达式分区可以替代列表分区。以下表格显示了两者之间的具体比较：

| 分区方法               | **列表分区**                                                 | **表达式分区**                                                 |
| ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 语法                   | `PARTITION BY LIST (partition_columns)（    PARTITION <partition_name> VALUES IN (value_list)    [, ...] )` | `PARTITION BY <partition_columns>`                            |
| 分区中每个分区列的多个值 | 支持。一个分区可以存储每个分区列的不同值的数据。在以下示例中，当加载的数据包含 `city` 列中的值 `Los Angeles`、`San Francisco` 和 `San Diego` 时，所有数据存储在一个分区中。 `pCalifornia`.`PARTITION BY LIST (city) (    PARTITION pCalifornia VALUES IN ("Los Angeles","San Francisco","San Diego")    [, ...] )` | 不支持。一个分区仅存储具有相同值的分区列的数据。例如，在表达式分区中使用了表达式 `PARTITION BY (city)`。当加载的数据包含 `city` 列中的值 `Los Angeles`、`San Francisco` 和 `San Diego` 时，StarRocks 将自动创建 `pLosAngeles`、`pSanFrancisco` 和 `pSanDiego` 三个分区。这三个分区分别存储了 `city` 列中的值 `Los Angeles`、`San Francisco` 和 `San Diego` 的数据。 |
| 创建数据加载前的分区   | 支持。需要在表创建时创建分区。                               | 不需要。数据加载期间可以自动创建分区。                     |
| 在数据加载期间自动创建列表分区 | 不支持。如果在数据加载期间与数据对应的分区不存在，则返回错误。 | 支持。如果在数据加载期间与数据对应的分区不存在，StarRocks 将自动创建该分区以存储数据。每个分区仅能包含分区列相同值的数据。 |
| SHOW CREATE TABLE语句   | 返回了创建表语句中的分区定义。                              | 在数据加载后，该语句返回使用了创建表语句中的分区子句的结果，即 `PARTITION BY (partition_columns)`。但是，返回的结果不显示任何自动创建的分区。如果需要查看自动创建的分区，执行 `SHOW PARTITIONS FROM <table_name>`。 |

## 用法

### 语法

```sql
PARTITION BY LIST (partition_columns)（
    PARTITION <partition_name> VALUES IN (value_list)
    [, ...]
)

partition_columns::= 
    <column> [,<column> [, ...] ]

value_list ::=
    value_item [, value_item [, ...] ]

value_item ::=
    { <value> | ( <value> [, <value>, [, ...] ] ) }    
```

### 参数

| **参数**        | **必填** | **描述**                                                |
| --------------- | -------- | ------------------------------------------------------- |
| `partition_columns` | 是   | 分区列的名称。分区列的值可以是字符串（不支持 BINARY）、日期或日期时间、整数和布尔值。分区列允许 `NULL` 值。 |
| `partition_name`    | 是   | 分区名称。建议根据业务场景设置适当的分区名称，以区分不同分区中的数据。 |
| `value_list`        | 是   | 分区中的分区列值列表。                                         |

### 示例

示例 1：假设您经常根据州或城市的数据中心计费详细信息进行查询。在表创建时，您可以指定分区列为 `city`，并指定每个分区存储同一州内的城市的数据。这种方法可以加快对特定州或城市的查询，并便于数据管理。

```SQL
CREATE TABLE t_recharge_detail1 (
    id bigint,
    user_id bigint,
    recharge_money decimal(32,2), 
    city varchar(20) not null,
    dt varchar(20) not null
)
DUPLICATE KEY(id)
PARTITION BY LIST (city) (
   PARTITION pLos_Angeles VALUES IN ("Los Angeles"),
   PARTITION pSan_Francisco VALUES IN ("San Francisco")
)
DISTRIBUTED BY HASH(`id`);
```

示例 2：假设您经常根据时间范围和特定州或城市的数据中心计费详细信息进行查询。在表创建期间，您可以指定分区列为 `dt` 和 `city`。这样，特定日期和特定州或城市的数据存储在同一个分区中，提高查询速度并便于数据管理。

```SQL
CREATE TABLE t_recharge_detail4 (
    id bigint,
    user_id bigint,
    recharge_money decimal(32,2), 
    city varchar(20) not null,
    dt varchar(20) not null
) ENGINE=OLAP
DUPLICATE KEY(id)
PARTITION BY LIST (dt,city) (
   PARTITION p202204_California VALUES IN (
       ("2022-04-01", "Los Angeles"),
       ("2022-04-01", "San Francisco"),
       ("2022-04-02", "Los Angeles"),
       ("2022-04-02", "San Francisco")
    ),
   PARTITION p202204_Texas VALUES IN (
       ("2022-04-01", "Houston"),
       ("2022-04-01", "Dallas"),
       ("2022-04-02", "Houston"),
       ("2022-04-02", "Dallas")
   )
)
DISTRIBUTED BY HASH(`id`);
```

## 限制

- 列表分区不支持动态分区和一次创建多个分区。
- 当使用 `ALTER TABLE <table_name> DROP PARTITION <partition_name>;` 语句删除使用列表分区创建的分区时，该分区中的数据将被直接删除，无法恢复。
- 目前不支持[备份和恢复](../administration/Backup_and_restore.md)通过列表分区创建的分区。
- 目前，StarRocks 不支持使用列表分区策略创建[异步物化视图](../using_starrocks/Materialized_view.md)的基表。