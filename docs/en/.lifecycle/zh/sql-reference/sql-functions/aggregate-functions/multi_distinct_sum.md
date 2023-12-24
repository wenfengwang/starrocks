---
displayed_sidebar: English
---

# multi_distinct_sum

## 描述

返回`expr`中不重复值的总和，相当于sum(distinct expr)。

## 语法

```Haskell
multi_distinct_sum(expr)
```

## 参数

- `expr`：参与计算的列。列值可以是以下类型：TINYINT、SMALLINT、INT、LARGEINT、FLOAT、DOUBLE或DECIMAL。

## 返回值

列值和返回值类型的映射关系如下：

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

## 例子

1. 创建一个具有`k0`作为INT字段的表。

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

2. 插入表中的值。

    ```sql
    -- 
    INSERT INTO tabl VALUES ('0'), ('1'), ('1'), ('1'), ('2'), ('2');
    ```

3. 使用multi_distinct_sum()计算`k0`列中不重复值的总和。

    ```plain text
    MySQL > select multi_distinct_sum(k0) from tabl;
    +------------------------+
    | multi_distinct_sum(k0) |
    +------------------------+
    |                      3 |
    +------------------------+
    1 row in set (0.03 sec)
    ```

    `k0`的不同值是0、1、2，将它们相加后我们可以得到3。