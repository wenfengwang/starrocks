---
displayed_sidebar: Chinese
---

# percentile_union

## 機能

グループ化された結果に対して集約を行うために使用されます。

## 文法

```sql
percentile_union(expr);
```

## パラメータ説明

`expr`: サポートされるデータ型は PERCENTILE です。

## 戻り値の説明

戻り値のデータ型は PERCENTILE です。

## 例

percentile物化ビューの使用例。

```sql
CREATE TABLE sales_records(
    record_id int, 
    seller_id int, 
    store_id int, 
    sale_date date, 
    sale_amt bigint
)
DISTRIBUTED BY hash(record_id) 
PROPERTIES ("replication_num" = "3");
```

`sale_amt` に対して PERCENTILE 型の物化ビューを作成します。

```sql
CREATE MATERIALIZED VIEW mv AS
SELECT store_id, percentile_union(percentile_hash(sale_amt)) FROM sales_records GROUP BY store_id;
```

PERCENTILE 型を含む集約テーブルを作成します。

```sql
CREATE TABLE sales_records(
    record_id int, 
    seller_id int, 
    store_id int, 
    sale_amt_per percentile
) ENGINE=OLAP
AGGREGATE KEY(`record_id`, `seller_id`, `store_id`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`record_id`)
PROPERTIES (
"replication_num" = "3",
"storage_format" = "DEFAULT"
);
```

PERCENTILE 型の列をクエリします。

```sql
select percentile_approx_raw(percentile_union(sale_amt_per), 0.99) from sales_records;
```

PERCENTILE を含む集約テーブルにデータをインポートします。

```sql
curl --location-trusted -u root -H "columns: record_id, seller_id, store_id,tmp, sale_amt_per = percentile_hash(tmp)" -H "column_separator:," -T a http://ip:port/api/test/sales_records/_stream_load
```
