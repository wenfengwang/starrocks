---
displayed_sidebar: English
---

# 列表分区

自v3.1起，StarRocks支持列表分区。数据根据每个分区预先定义的值列表进行分区，这样可以加快基于枚举值的查询速度，并便于管理。

## 介绍

您需要在每个分区中明确指定列值列表。这些值不必像范围分区中所要求的连续时间或数字范围那样连续。在数据加载时，StarRocks会根据数据分区列的值与各分区预定义列值之间的映射，将数据存储到相应的分区中。

![list_partitioning](../assets/list_partitioning.png)

列表分区适用于存储列值为少数枚举值的数据，您通常会根据这些枚举值进行查询和管理。例如，列可能代表地理位置、州或分类，每个值代表一个独立的分类。通过基于枚举值分区数据，您可以提升查询效率和简化数据管理。

**列表分区特别适合于每个分区列需要包含多个值的场景**。例如，如果一个表包含代表个人所在城市的`City`列，并且您常常需要按照州和城市来查询和管理数据。在创建表时，您可以将`City`列设为列表分区的分区列，并指定同一个州内不同城市的数据存放在同一分区，例如 `PARTITION pCalifornia VALUES IN ("Los Angeles", "San Francisco", "San Diego")`。这种方法可以加快基于州和城市的查询，同时便于数据管理。

如果一个分区只需要包含每个分区列相同值的数据，建议使用[表达式分区](./expression_partitioning.md)。

**列表分区**与**[表达式分区](./expression_partitioning.md)**的比较

列表分区与表达式分区（推荐）的主要区别在于，列表分区需要您手动逐一创建分区。而表达式分区在数据加载时可以自动创建分区，简化了分区过程。在大多数情况下，表达式分区能够替代列表分区。以下表格展示了两者的具体对比：

|分区方式|列表分区|表达式分区|
|---|---|---|
|语法|PARTITION BY LIST (partition_columns)（ PARTITION <partition_name> VALUES IN (value_list) [, ...] )|PARTITION BY <partition_columns>|
|分区中每个分区列的多个值|支持。分区可以在每个分区列中存储具有不同值的数据。在下面的示例中，当加载的数据的城市列中包含洛杉矶、旧金山和圣地亚哥时，所有数据都存储在一个分区中。 pCalifornia.PARTITION BY LIST (city) ( PARTITION pCalifornia VALUES IN ("Los Angeles","San Francisco","San Diego") [, ...] )|不支持。分区在分区列中存储具有相同值的数据。例如，表达式分区中使用表达式PARTITION BY (city)。当加载的数据在城市列中包含洛杉矶、旧金山和圣地亚哥时，StarRocks 将自动创建三个分区 pLosAngeles、pSanFrancisco 和 pSanDiego。三个分区分别存储城市列中值为“Los Angeles”、“San Francisco”和“San Diego”的数据。|
|数据加载前创建分区|支持。分区需要在创建表时创建。|不需要这样做。数据加载时可以自动创建分区。|
|数据加载期间自动创建List分区|不支持。加载数据时如果数据对应的分区不存在，则返回错误。|支持。如果数据加载时数据对应的分区不存在，StarRocks会自动创建该分区来存储数据。每个分区只能包含分区列具有相同值的数据。|
|SHOW CREATE TABLE|返回CREATE TABLE语句中的分区定义。|加载数据后，语句会使用CREATE TABLE语句中使用的分区子句，即PARTITION BY (partition_columns)返回结果。但返回的结果并没有显示任何自动创建的分区。如果需要查看自动创建的分区，执行SHOW PARTITIONS FROM <table_name>.|

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

|参数|参数|说明|
|---|---|---|
|partition_columns|YES|分区列的名称。分区列值可以是字符串（不支持 BINARY）、日期或日期时间、整数和布尔值。分区列允许 NULL 值。|
|partition_name|YES|分区名称。建议根据业务场景设置合适的分区名称，以区分不同分区的数据。|
|value_list|YES|分区中分区列值的列表。|

### 示例

示例1：假设您经常需要根据州或城市查询数据中心的计费详情。在创建表时，您可以将分区列设置为city，并指定每个分区存储同一州内的城市数据。这种方法可以加速对特定州或城市的查询，并简化数据管理。

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

示例2：假设您经常需要根据时间范围和特定的州或城市查询数据中心的计费详情。在创建表时，您可以将分区列设置为dt和city。这样，特定日期和特定州或城市的数据就可以存储在同一个分区中，提高了查询速度并便于数据管理。

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
- 目前，StarRocks的[共享数据模式](../deployment/shared_data/s3.md)不支持这个功能。
- 使用ALTER TABLE <table_name> DROP PARTITION <partition_name>;语句删除通过列表分区创建的分区时，该分区中的数据会被直接移除且无法恢复。
- 目前，您无法[备份和恢复](../administration/Backup_and_restore.md)通过列表分区创建的分区。
- 目前，StarRocks不支持创建[异步物化视图](../using_starrocks/Materialized_view.md)以及使用列表分区策略创建的基表。
