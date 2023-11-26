---
displayed_sidebar: "Japanese"
---

# multi_distinct_sum

## 説明

`expr` の重複を除いた値の合計を返します。sum(distinct expr) と同等です。

## 構文

```Haskell
multi_distinct_sum(expr)
```

## パラメータ

- `expr`: 計算に関与する列。列の値は以下の型である必要があります: TINYINT、SMALLINT、INT、LARGEINT、FLOAT、DOUBLE、または DECIMAL。

## 戻り値

列の値と戻り値の型のマッピングは以下の通りです:

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

1. `k0` を INT フィールドとするテーブルを作成します。

    ```sql
    CREATE TABLE tabl
    (k0 BIGINT NOT NULL) ENGINE=OLAP
    DUPLICATE KEY(`k0`)
    COMMENT "OLAP"
    DISTRIBUTED BY HASH(`k0`)
    PROPERTIES(
        "replication_num" = "3",
        "storage_format" = "DEFAULT"
    );
    ```

2. テーブルに値を挿入します。

    ```sql
    -- 
    INSERT INTO tabl VALUES ('0'), ('1'), ('1'), ('1'), ('2'), ('2');
    ```

3. multi_distinct_sum() を使用して、`k0` 列の重複を除いた値の合計を計算します。

    ```plain text
    MySQL > select multi_distinct_sum(k0) from tabl;
    +------------------------+
    | multi_distinct_sum(k0) |
    +------------------------+
    |                      3 |
    +------------------------+
    1 row in set (0.03 sec)
    ```

    `k0` の重複を除いた値は 0、1、2 であり、それらを合計すると 3 になります。
