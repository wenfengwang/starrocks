---
displayed_sidebar: English
---

# percentile_union

## 説明

PERCENTILE データを集約します。

## 構文

```sql
percentile_union(expr);
```

## パラメーター

`expr`: サポートされるデータ型は PERCENTILE です。

## 戻り値

PERCENTILE 値を返します。

## 例

例 1: マテリアライズドビューでパーセンタイルデータを使用します。

テーブルを作成します。

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

テーブルの `sale_amt` 列を基づいてマテリアライズドビューを作成します。

```sql
CREATE MATERIALIZED VIEW mv AS
SELECT store_id, percentile_union(percentile_hash(sale_amt))
FROM sales_records
GROUP BY store_id;
```

例 2: PERCENTILE データをロードします。

PERCENTILE 列 `sale_amt_per` を含む集約テーブルを作成します。

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

`sale_amt_per` からデータをクエリします。

```sql
SELECT percentile_approx_raw(PERCENTILE_UNION(sale_amt_per), 0.99) FROM sales_records;
```

PERCENTILE データを `sales_records` テーブルにロードします。

```sql
curl --location-trusted -u root
    -H "columns: record_id, seller_id, store_id, tmp, sale_amt_per=percentile_hash(tmp)"
    -H "column_separator:,"
    -T a http://<ip:port>/api/test/sales_records/_stream_load
```
