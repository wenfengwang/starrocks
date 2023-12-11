---
displayed_sidebar: "Japanese"
---

# VARCHAR（文字列）

## 説明

VARCHAR（M）

可変長の文字列。`M`は文字列の長さを示します。デフォルト値は`1`です。単位：バイト。

- StarRocks 2.1より前のバージョンでは、`M`の値の範囲は1～65533です。
- [プレビュー] StarRocks 2.1およびそれ以降のバージョンでは、`M`の値の範囲は1～1048576です。

## 例

テーブルを作成し、列のタイプをVARCHARと指定します。

```SQL
CREATE TABLE varcharDemo (
    pk INT COMMENT "range [-2147483648, 2147483647]",
    pd_type VARCHAR(20) COMMENT "range char(m),m in (1-65533) "
) ENGINE=OLAP 
DUPLICATE KEY(pk)
COMMENT "OLAP"
DISTRIBUTED BY HASH(pk)
```