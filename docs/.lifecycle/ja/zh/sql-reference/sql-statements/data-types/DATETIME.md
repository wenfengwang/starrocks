---
displayed_sidebar: Chinese
---

# DATETIME

## 説明

日付時間型で、値の範囲は ['0000-01-01 00:00:00', '9999-12-31 23:59:59'] です。

表示形式は 'YYYY-MM-DD HH:MM:SS' です。

## 例

テーブル作成時にカラムの型を DATETIME として指定します。

```sql
CREATE TABLE dateTimeDemo (
    pk INT COMMENT "範囲 [-2147483648, 2147483647]",
    relTime DATETIME COMMENT "YYYY-MM-DD HH:MM:SS"
) ENGINE=OLAP 
DUPLICATE KEY(pk)
COMMENT "OLAP"
DISTRIBUTED BY HASH(pk);
```
