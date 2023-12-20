---
displayed_sidebar: English
---

# 多重唯一值求和

## 描述

返回表达式中唯一值的总和，等同于 sum(distinct expr)。

## 语法

```Haskell
multi_distinct_sum(expr)
```

## 参数

- expr：参与计算的列。列值可以是以下数据类型：TINYINT、SMALLINT、INT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL。

## 返回值

列值与返回值类型之间的对应关系如下：

- BOOLEAN -> BIGINT
- TINYINT -> BIGINT
- SMALLINT -> BIGINT
- INT -> BIGINT
- LARGEINT -> LARGEINT
- FLOAT -> DOUBLE
- DOUBLE -> DOUBLE
- LARGEINT -> LARGEINT
- DECIMALV2 -> DECIMALV2
- DECIMAL32 -> DECIMAL128
- DECIMAL64 -> DECIMAL128
- DECIMAL128 -> DECIMAL128

## 示例

1. 创建一个表，k0 是 INT 类型字段。

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

2. 向表中插入数据。

   ```sql
   -- 
   INSERT INTO tabl VALUES ('0'), ('1'), ('1'), ('1'), ('2'), ('2');
   ```

3. 使用 multi_distinct_sum() 函数来计算 k0 列不同值的总和。

   ```plain
   MySQL > select multi_distinct_sum(k0) from tabl;
   +------------------------+
   | multi_distinct_sum(k0) |
   +------------------------+
   |                      3 |
   +------------------------+
   1 row in set (0.03 sec)
   ```

   k0 的唯一值有 0、1、2，加起来的结果是 3。
