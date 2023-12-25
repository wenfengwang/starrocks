---
displayed_sidebar: Chinese
---

# multi_distinct_sum

## 機能

`expr` から重複する値を除いた合計を返します。これは sum(distinct expr) と同等です。

## 文法

```Haskell
multi_distinct_sum(expr)
```

## パラメータ説明

`expr`: 計算に使用する列を指定します。列の値は TINYINT、SMALLINT、INT、LARGEINT、FLOAT、DOUBLE、または DECIMAL 型である可能性があります。

## 戻り値の説明

列の値と戻り値の型のマッピングは以下の通りです：

- BOOLEAN -> BIGINT
- TINYINT -> BIGINT
- SMALLINT -> BIGINT
- INT -> BIGINT
- BIGINT -> BIGINT
- FLOAT -> DOUBLE
- DOUBLE -> DOUBLE
- LARGEINT -> LARGEINT
- DECIMALV2 -> DECIMALV2
- DECIMAL32 -> DECIMAL128
- DECIMAL64 -> DECIMAL128
- DECIMAL128 -> DECIMAL128

## 例

```plain text
-- int 型のフィールドを1つだけ持つテーブルを作成します。
CREATE TABLE tabl
(k0 BIGINT NOT NULL) ENGINE=OLAP
DUPLICATE KEY(`k0`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`k0`)
PROPERTIES(
    "replication_num" = "3",
    "storage_format" = "DEFAULT"
);
Query OK, 0 rows affected (0.01 sec)

-- 値 0,1,1,1,2,2 を挿入します。
MySQL > INSERT INTO tabl VALUES ('0'), ('1'), ('1'), ('1'), ('2'), ('2');
Query OK, 6 rows affected (0.15 sec)

-- k0 の重複を除いた値は 0,1,2 で、合計すると 3 になります。
MySQL > select multi_distinct_sum(k0) from tabl;
+------------------------+
| multi_distinct_sum(k0) |
+------------------------+
|                      3 |
+------------------------+
1 row in set (0.03 sec)
```
