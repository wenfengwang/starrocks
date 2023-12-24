---
displayed_sidebar: English
---

# percentile_empty

## 描述

构造一个 PERCENTILE 值，用于在使用 [Stream Load](../../../loading/StreamLoad.md) 或 [INSERT INTO](../../../loading/InsertInto.md) 加载数据时填充空值。

## 语法

```Haskell
PERCENTILE_EMPTY();
```

## 参数

无

## 返回值

返回 PERCENTILE 值。

## 例子

创建一个表。`percent` 列是 PERCENTILE 列。

```sql
CREATE TABLE `aggregate_tbl` (
  `site_id` largeint(40) NOT NULL COMMENT "站点ID",
  `date` date NOT NULL COMMENT "事件时间",
  `city_code` varchar(20) NULL COMMENT "用户城市代码",
  `pv` bigint(20) SUM NULL DEFAULT "0" COMMENT "总页面浏览量",
  `percent` PERCENTILE PERCENTILE_UNION COMMENT "其他"
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