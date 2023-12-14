---
displayed_sidebar: "Chinese"
---

# multi_distinct_sum

## 描述

返回 `expr` 中不同值的总和，等同于 sum(distinct expr)。

## 语法

```Haskell
multi_distinct_sum(expr)
```

## 参数

- `expr`: 参与计算的列。列的值可以是以下类型之一：TINYINT、SMALLINT、INT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL。

## 返回值

列值与返回值类型的映射如下：

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

## 示例

1. 创建一个以 `k0` 为 INT 字段的表。

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

2. 向表中插入值。

    ```sql
    -- 
    INSERT INTO tabl VALUES ('0'), ('1'), ('1'), ('1'), ('2'), ('2');
    ```

3. 使用 multi_distinct_sum() 计算 `k0` 列中不同值的总和。

    ```plain text
    MySQL > select multi_distinct_sum(k0) from tabl;
    +------------------------+
    | multi_distinct_sum(k0) |
    +------------------------+
    |                      3 |
    +------------------------+
    1 row in set (0.03 sec)
    ```

    `k0` 的不同值为 0、1、2，相加后得到 3。