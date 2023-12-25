---
displayed_sidebar: Chinese
---

# SMALLINT

## 説明

2バイトの符号付き整数で、範囲は [-32768, 32767] です。

## 例

テーブル作成時にカラムの型を SMALLINT として指定します。

```sql
CREATE TABLE smallintDemo (
    pk SMALLINT COMMENT "range [-32768, 32767]"
) ENGINE=OLAP 
DUPLICATE KEY(pk)
COMMENT "OLAP"
DISTRIBUTED BY HASH(pk);
```
