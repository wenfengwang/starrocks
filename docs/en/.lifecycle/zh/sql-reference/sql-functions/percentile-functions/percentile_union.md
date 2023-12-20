---
displayed_sidebar: English
---

# percentile_union

## 描述

聚合 PERCENTILE 数据。

## 语法

```sql
percentile_union(expr);
```

## 参数

`expr`：支持的数据类型是 PERCENTILE。

## 返回值

返回一个 PERCENTILE 值。

## 示例

示例 1：在物化视图中使用百分位数据。

创建一个表。

```sql
CREATE TABLE sales_records(
    record_id int, 
    seller_id int, 
    store_id int, 
    sale_date date, 
    sale_amt bigint
) DISTRIBUTED BY HASH(record_id) 
PROPERTIES ("replication_num" = "3");
```

根据表的 `sale_amt` 列创建物化视图。

```sql
CREATE MATERIALIZED VIEW mv AS
SELECT store_id, percentile_union(percentile_hash(sale_amt))
FROM sales_records
GROUP BY store_id;
```

示例 2：加载 PERCENTILE 数据。

创建一个包含 PERCENTILE 列 `sale_amt_per` 的聚合表。

```sql
CREATE TABLE sales_records(
    record_id int, 
    seller_id int, 
    store_id int, 
    sale_amt_per PERCENTILE PERCENTILE_UNION
) ENGINE=OLAP
AGGREGATE KEY(`record_id`, `seller_id`, `store_id`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`record_id`)
PROPERTIES (
    "replication_num" = "3",
    "storage_format" = "DEFAULT"
);
```

从 `sale_amt_per` 查询数据。

```sql
SELECT percentile_approx_raw(PERCENTILE_UNION(sale_amt_per), 0.99) FROM sales_records;
```

将包含 PERCENTILE 数据的数据加载到 `sales_records` 表中。

```sql
curl --location-trusted -u root
    -H "columns: record_id, seller_id, store_id, tmp, sale_amt_per=percentile_hash(tmp)"
    -H "column_separator:,"
    -T a http://<ip:port>/api/test/sales_records/_stream_load
```