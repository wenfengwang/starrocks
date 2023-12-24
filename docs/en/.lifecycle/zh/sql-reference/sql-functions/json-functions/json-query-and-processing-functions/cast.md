---
displayed_sidebar: English
---

# 强制转换

## 描述

在 JSON 类型和 SQL 类型之间转换值。

## 语法

- 从 JSON 到 SQL 的转换

```Haskell
cast(json_expr AS sql_data_type)
```

- 从 SQL 到 JSON 的转换

```Haskell
cast(sql_expr AS JSON)
```

## 参数

- `json_expr`：表示要转换为 SQL 值的 JSON 值的表达式。

- `sql_data_type`：要将 JSON 值转换为的 SQL 数据类型。仅支持 STRING、VARCHAR、CHAR、BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、DOUBLE 和 FLOAT 数据类型。

- `sql_expr`：表示要转换为 JSON 值的 SQL 值的表达式。此参数支持 `sql_data_type` 参数支持的所有 SQL 数据类型。

## 返回值

- 如果使用 `cast(json_expr AS sql_data_type)` 语法，则强制转换函数将返回由 `sql_data_type` 参数指定的 SQL 数据类型的值。

- 如果使用 `cast(sql_expr AS JSON)` 语法，则强制转换函数将返回一个 JSON 值。

## 使用说明

- 从 SQL 到 JSON 的转换

  - 如果 SQL 值超过 JSON 支持的精度，则强制转换函数将返回 `NULL` 以防止算术溢出。

  - 如果 SQL 值为 `NULL`，则强制转换函数不会将 SQL 值 `NULL` 转换为 JSON 值 `NULL`。返回值仍为 SQL 值 `NULL`。

- 从 JSON 到 SQL 的转换

  - 强制转换函数仅支持兼容的 JSON 和 SQL 数据类型之间的转换。例如，您可以将 JSON 字符串转换为 SQL 字符串。

  - 强制转换函数不支持不兼容的 JSON 和 SQL 数据类型之间的转换。例如，如果将 JSON 数字转换为 SQL 字符串，则该函数将返回 `NULL`。

  - 如果发生算术溢出，则强制转换函数将返回 SQL 值 `NULL`。

  - 如果将 JSON `NULL` 值转换为 SQL 值，则该函数将返回 SQL 值 `NULL`。

  - 如果将 JSON 字符串转换为 VARCHAR 值，该函数将返回一个未用双引号（"）括起来的 VARCHAR 值。

## 例子

示例1：将 JSON 值转换为 SQL 值。

```plaintext
-- 将 JSON 值转换为 INT 值。
mysql> select cast(parse_json('{"a": 1}') -> 'a' as int);
+--------------------------------------------+
| CAST((parse_json('{"a": 1}')->'a') AS INT) |
+--------------------------------------------+
|                                          1 |
+--------------------------------------------+

-- 将 JSON 字符串转换为 VARCHAR 值。
mysql> select cast(parse_json('"star"') as varchar);
+---------------------------------------+
| cast(parse_json('"star"') AS VARCHAR) |
+---------------------------------------+
| star                                  |
+---------------------------------------+

-- 将 JSON 对象转换为 VARCHAR 值。
mysql> select cast(parse_json('{"star": 1}') as varchar);
+--------------------------------------------+
| cast(parse_json('{"star": 1}') AS VARCHAR) |
+--------------------------------------------+
| {"star": 1}                                |
+--------------------------------------------+

-- 将 JSON 数组转换为 VARCHAR 值。
mysql> select cast(parse_json('[1,2,3]') as varchar);
+----------------------------------------+
| cast(parse_json('[1,2,3]') AS VARCHAR) |
+----------------------------------------+
| [1, 2, 3]                              |
+----------------------------------------+
```

示例2：将 SQL 值转换为 JSON 值。

```plaintext
-- 将 INT 值转换为 JSON 值。
mysql> select cast(1 as json);
+-----------------+
| cast(1 AS JSON) |
+-----------------+
| 1               |
+-----------------+

-- 将 VARCHAR 值转换为 JSON 值。
mysql> select cast("star" as json);
+----------------------+
| cast('star' AS JSON) |
+----------------------+
| "star"               |
+----------------------+

-- 将 BOOLEAN 值转换为 JSON 值。
mysql> select cast(true as json);
+--------------------+
| cast(TRUE AS JSON) |
+--------------------+
| true               |
+--------------------+
```