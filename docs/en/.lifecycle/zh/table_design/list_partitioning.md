---
displayed_sidebar: English
---

# 列表分区

自 v3.1 起，StarRocks 支持列表分区。数据根据每个分区预定义的值列表进行分区，这可以加快基于枚举值的查询并便于管理。

## 介绍

您需要在每个分区中明确指定列值列表。这些值不需要像范围分区所需的时间或数字范围那样连续。在数据加载过程中，StarRocks 会根据数据分区列的值与每个分区预定义列值之间的映射，将数据存储在相应的分区中。

![list_partitioning](../assets/list_partitioning.png)

列表分区适用于存储列包含少量枚举值的数据，您经常需要根据这些枚举值查询和管理数据。例如，列可能代表地理位置、状态或类别，每个值代表一个独立的类别。通过基于枚举值分区，您可以提升查询性能并简化数据管理。

**列表分区特别适用于每个分区列需要包含多个值的场景**。例如，如果一个表包含代表个人所在城市的 `City` 列，并且您经常需要按州和城市查询和管理数据。在创建表时，您可以将 `City` 列作为列表分区的分区列，并指定同一州内不同城市的数据放在同一个分区，例如 `PARTITION pCalifornia VALUES IN ("Los Angeles", "San Francisco", "San Diego")`。这种方式可以加快基于州和城市的查询，同时便于数据管理。

如果分区只需包含每个分区列相同值的数据，建议使用[表达式分区](./expression_partitioning.md)。

**列表分区与[表达式分区](./expression_partitioning.md)的比较**

列表分区与表达式分区（推荐）的主要区别在于，列表分区需要您手动逐一创建分区。而表达式分区可以在数据加载过程中自动创建分区，简化了分区操作。在大多数情况下，表达式分区可以替代列表分区。两者的具体对比如下表所示：

|分区方法|**列表分区**|**表达式分区**|
|---|---|---|
|语法|`PARTITION BY LIST (partition_columns)（ PARTITION <partition_name> VALUES IN (value_list) [, ...] )`|`PARTITION BY <partition_columns>`|
|分区中每个分区列的多个值|支持。一个分区可以存储每个分区列不同值的数据。例如，当加载的数据在 `city` 列中包含 `Los Angeles`、`San Francisco` 和 `San Diego` 时，所有数据都存储在 `pCalifornia` 分区中。`PARTITION BY LIST (city) ( PARTITION pCalifornia VALUES IN ("Los Angeles", "San Francisco", "San Diego") [, ...] )`|不支持。一个分区只存储分区列中相同值的数据。例如，在表达式分区中使用 `PARTITION BY (city)`。当加载的数据在 `city` 列中包含 `Los Angeles`、`San Francisco` 和 `San Diego` 时，StarRocks 会自动创建三个分区 `pLosAngeles`、`pSanFrancisco` 和 `pSanDiego`。这三个分区分别存储 `city` 列中值为 `Los Angeles`、`San Francisco` 和 `San Diego` 的数据。|
|数据加载前创建分区|支持。需要在创建表时创建分区。|不需要。数据加载时可以自动创建分区。|
|数据加载期间自动创建列表分区|不支持。如果加载数据时不存在对应的分区，则返回错误。|支持。如果加载数据时不存在对应的分区，StarRocks 会自动创建该分区来存储数据。每个分区只能包含分区列中相同值的数据。|
|SHOW CREATE TABLE|返回 `CREATE TABLE` 语句中的分区定义。|数据加载后，语句返回带有 `CREATE TABLE` 语句中的分区子句的结果，即 `PARTITION BY (partition_columns)`。但返回结果不包括任何自动创建的分区。若需查看自动创建的分区，请执行 `SHOW PARTITIONS FROM <table_name>`。|

## 使用方法

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
    { <value> | ( <value> [, <value> [, ...] ] ) }    
```

### 参数

|**参数**|**必填**|**描述**|
|---|---|---|
|`partition_columns`|是|分区列的名称。分区列的值可以是字符串（不支持 BINARY）、日期或 datetime、整数和布尔值。分区列允许 `NULL` 值。|
|`partition_name`|是|分区名称。建议根据业务场景设置合适的分区名称，以区分不同分区中的数据。|
|`value_list`|是|分区中分区列值的列表。|

### 示例

示例 1：假设您经常需要根据州或城市查询数据中心计费的详细信息。在创建表时，您可以将分区列设置为 `city`，并指定每个分区存储同一州内的城市数据。这种方法可以加快对特定州或城市的查询，并简化数据管理。

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

示例 2：假设您经常需要根据时间范围和特定州或城市查询数据中心计费的详细信息。在创建表时，您可以将分区列设置为 `dt` 和 `city`。这样，特定日期和特定州或城市的数据就会被存储在同一个分区中，从而提高查询速度并简化数据管理。

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

- 列表分区支持动态分区和一次性创建多个分区。
- 目前，StarRocks 的 [共享数据模式](../deployment/shared_data/s3.md) 不支持此功能。
- 使用 `ALTER TABLE <table_name> DROP PARTITION <partition_name>;` 语句删除使用列表分区创建的分区时，该分区中的数据将被直接移除且无法恢复。
- 目前无法对使用列表分区创建的分区进行[备份和恢复](../administration/Backup_and_restore.md)。
- 目前，StarRocks 不支持对使用列表分区策略创建的基表创建[异步物化视图](../using_starrocks/Materialized_view.md)。