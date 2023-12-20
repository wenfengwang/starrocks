---
displayed_sidebar: English
---

# 百分位空值填充

## 描述

构造一个 PERCENTILE 值，用于在使用[Stream Load](../../../loading/StreamLoad.md)或[INSERT INTO](../../../loading/InsertInto.md)加载数据时填充空值。

## 语法

```Haskell
PERCENTILE_EMPTY();
```

## 参数

无

## 返回值

返回一个 PERCENTILE 值。

## 示例

创建一张表。该表的百分比列为 PERCENTILE 类型。

```sql
CREATE TABLE `aggregate_tbl` (
  `site_id` largeint(40) NOT NULL COMMENT "id of site",
  `date` date NOT NULL COMMENT "time of event",
  `city_code` varchar(20) NULL COMMENT "city_code of user",
  `pv` bigint(20) SUM NULL DEFAULT "0" COMMENT "total page views",
  `percent` PERCENTILE PERCENTILE_UNION COMMENT "others"
) ENGINE=OLAP
AGGREGATE KEY(`site_id`, `date`, `city_code`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`site_id`)
PROPERTIES ("replication_num" = "3");
```

向表中插入数据。

```sql
INSERT INTO aggregate_tbl VALUES
(5, '2020-02-23', 'city_code', 555, percentile_empty());
```
