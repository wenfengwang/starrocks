---
displayed_sidebar: English
---

# 百分位数合并

## 描述

聚合百分位数（PERCENTILE）数据。

## 语法

```sql
percentile_union(expr);
```

## 参数

expr：支持的数据类型为百分位数（PERCENTILE）。

## 返回值

返回一个百分位数（PERCENTILE）值。

## 示例

示例 1：在物化视图中使用百分位数数据。

创建一张表。

```sql
CREATE TABLE sales_records(
    record_id int, 
    seller_id int, 
    store_id int, 
    sale_date date, 
    sale_amt bigint
) distributed BY hash(record_id) 
PROPERTIES ("replication_num" = "3");
```

基于该表的 sale_amt 列创建一个物化视图。

```sql
create materialized view mv as
select store_id, percentile_union(percentile_hash(sale_amt))
from sales_records
group by store_id;
```

示例 2：加载百分位数数据。

创建一个含有百分位数列 sale_amt_per 的聚合表。

```sql
CREATE TABLE sales_records(
    record_id int, 
    seller_id int, 
    store_id int, 
    sale_amt_per percentile percentile_union
) ENGINE=OLAP
AGGREGATE KEY(`record_id`, `seller_id`, `store_id`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`record_id`)
PROPERTIES (
    "replication_num" = "3",
    "storage_format" = "DEFAULT"
);
```

从 sale_amt_per 列查询数据。

```sql
select percentile_approx_raw(percentile_union(sale_amt_per), 0.99) from sales_records;
```

将含有百分位数数据的数据加载进 sales_records 表。

```sql
curl --location-trusted -u root
    -H "columns: record_id, seller_id, store_id,tmp, sale_amt_per =percentile_hash(tmp)"
    -H "column_separator:,"
    -T a http://<ip:port>/api/test/sales_records/_stream_load
```
