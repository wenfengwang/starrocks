---
displayed_sidebar: Chinese
---

# INT型

## 説明

4バイトの符号付き整数。範囲は [-2147483648, 2147483647] です。

## 例

テーブル作成時にカラムの型をINTとして指定します。

```sql
CREATE TABLE intDemo (
    pk INT COMMENT "範囲 [-2147483648, 2147483647]"
) ENGINE=OLAP 
DUPLICATE KEY(pk)
COMMENT "OLAP"
DISTRIBUTED BY HASH(pk);
```
