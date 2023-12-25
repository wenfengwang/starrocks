---
displayed_sidebar: Chinese
---

# BIGINT

## 説明

8バイトの符号付き整数。範囲は [-9223372036854775808, 9223372036854775807] です。

## 例

テーブルを作成する際に、フィールドの型を BIGINT として指定します。

```sql
CREATE TABLE bigIntDemo (
    pk BIGINT(20) NOT NULL COMMENT ""
) ENGINE=OLAP 
DUPLICATE KEY(pk)
COMMENT "OLAP"
DISTRIBUTED BY HASH(pk);
```
