---
displayed_sidebar: Chinese
---

# DATE

## 説明

日付型で、現在の取りうる範囲は ['0000-01-01', '9999-12-31'] で、デフォルトの表示形式は 'YYYY-MM-DD' です。

## 例

テーブル作成時にカラムの型を DATE として指定します。

```sql
CREATE TABLE dateDemo (
    pk INT COMMENT "range [-2147483648, 2147483647]",
    make_time DATE NOT NULL COMMENT "YYYY-MM-DD"
) ENGINE=OLAP 
DUPLICATE KEY(pk)
COMMENT "OLAP"
DISTRIBUTED BY HASH(pk);
```
