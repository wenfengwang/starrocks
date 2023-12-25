---
displayed_sidebar: Chinese
---

# CHAR

## 説明

CHAR(M)

固定長文字列で、M は固定長文字列の長さを表します。M の範囲は 1~255 です。

## 例

テーブル作成時にカラムの型を CHAR として指定します。

```sql
CREATE TABLE charDemo (
    pk INT COMMENT "range [-2147483648, 2147483647]",
    pd_type CHAR(20) NOT NULL COMMENT "range char(m),m in (1-255) "
) ENGINE=OLAP 
DUPLICATE KEY(pk)
COMMENT "OLAP"
DISTRIBUTED BY HASH(pk);
```
