---
displayed_sidebar: English
---

# 列表分区

从 v3.1 开始，StarRocks 支持列表分区。数据基于每个分区的预定义值列表进行分区，可以加快查询速度，方便根据枚举值进行管理。

## 介绍

您需要明确指定每个分区中的列值列表。这些值不需要是连续的，这与范围分区中所需的连续时间或数值范围不同。在数据加载过程中，StarRocks 会根据数据的分区列值与每个分区预定义的列值的映射关系，将数据存储在对应的分区中。

![list_partitioning](../assets/list_partitioning.png)

列表分区适用于存储列中包含少量枚举值的数据，并且经常根据这些枚举值进行查询和管理数据。例如，列表示地理位置、州、和类别。列中的每个值都表示一个独立的类别。通过根据枚举值对数据进行分区，可以提高查询性能并简化数据管理。

**列表分区对于分区需要为每个分区列包含多个值的方案特别有用**。例如，如果表包含 `City` 表示个人所在城市的列，并且您经常按州和城市查询和管理数据。在创建表时，您可以将`City`列用作列表分区的分区列，并指定将同一州内各个城市的数据放在一个分区中，例如 `PARTITION pCalifornia VALUES IN ("Los Angeles", "San Francisco", "San Diego")`。该方法可以加速基于州和城市的查询，同时促进数据管理。

如果一个分区只需要包含每个分区列值相同的数据，建议使用 [表达式分区](./expression_partitioning.md)。

**列表分区与表达式分区的比较 [表达式分区](./expression_partitioning.md)**

列表分区和表达式分区（推荐）的主要区别在于，列表分区需要您手动逐个创建分区。另一方面，表达式分区可以在加载过程中自动创建分区，以简化分区。在大多数情况下，表达式分区可以取代列表分区。两者的具体对比如下表所示：

| 分区方式                                      | **列表分区**                                        | **表达式分区**                                  |
| -------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 语法                                                   | `PARTITION BY LIST (partition_columns)（    PARTITION <partition_name> VALUES IN (value_list)    [, ...] )` | `PARTITION BY <partition_columns>`                           |
| 分区中每个分区列的多个值 | 支持。分区可以在每个分区列中存储具有不同值的数据。在以下示例中，当加载的数据包含值 `Los Angeles`、 `San Francisco`和 `San Diego` `city` 时，所有数据都存储在一个分区中。 `pCalifornia`。`PARTITION BY LIST (city) (    PARTITION pCalifornia VALUES IN ("Los Angeles","San Francisco","San Diego")    [, ...] )` | 不支持。分区将具有相同值的数据存储在分区列中 例如，表达式`PARTITION BY (city)`用于表达式分区。当加载的数据 `Los Angeles` 在列中`San Francisco`包含值 、 `San Diego`和 `city` 时，StarRocks 会自动创建三个分区 `pLosAngeles`、 `pSanFrancisco`和 `pSanDiego`。这三个分区分别将数据与值 `Los Angeles,` `San Francisco`和 `San Diego` 存储在列中 `city` 。 |
| 在数据加载之前创建分区                    | 支持。需要在创建表时创建分区。  | 没有必要这样做。可以在数据加载过程中自动创建分区。 |
| 在数据加载期间自动创建列表分区 | 不支持。如果在数据加载过程中不存在与数据对应的分区，则返回错误。 | 支持。如果在数据加载过程中不存在数据对应的分区，StarRocks 会自动创建该分区来存储数据。每个分区只能包含分区列具有相同值的数据。 |
| 显示创建表                                        | 在 CREATE TABLE 语句中返回分区定义。 | 加载数据后，该语句返回结果，其中包含 CREATE TABLE 语句中使用的 partition 子句，即 `PARTITION BY (partition_columns)`.但是，返回的结果不显示任何自动创建的分区。如果需要查看自动创建的分区，请执行 `SHOW PARTITIONS FROM <table_name>`. |

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

| **参数**      | **参数** | **描述**                                              |
| ------------------- | -------------- | ------------------------------------------------------------ |
| `partition_columns` | 是的            | 分区列的 ames。分区列值可以是字符串（不支持 BIARY）、日期或日期时间、整数和布尔值。分区列允许 `NULL` 值。 |
| `partition_name`    | 是的            | 分区名称。建议根据业务场景设置合适的分区名称，以区分不同分区中的数据。 |
| `value_list`        | 是的            | 分区中分区列值的列表。            |

### 例子

示例1：假设您经常查询基于州或城市的数据中心计费明细。在创建表时，您可以指定分区列， `city` 并指定每个分区存储相同状态下的城市数据。该方法可以加速对特定州或城市的查询，并促进数据管理。

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

示例2：假设您经常根据时间范围和特定州或城市查询数据中心计费明细。在表创建过程中，可以将分区列指定为 `dt`  和 `city`。这样，特定日期和特定州或城市的数据将存储到同一个分区中，从而提高了查询速度并方便了数据管理。

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

- 列表分区支持动态分区和一次创建多个分区。
- 目前，StarRocks 的[共享数据模式](../deployment/shared_data/s3.md)暂不支持此功能。
- 使用 `ALTER TABLE <table_name> DROP PARTITION <partition_name>;` 语句删除使用列表分区创建的分区时，该分区中的数据会被直接删除，并且无法恢复。
- 目前，您无法 [备份和还原](../administration/Backup_and_restore.md) 由列表分区创建的分区。
- 目前，StarRocks 不支持使用列表分区策略创建的基表创建异步物化视图。
