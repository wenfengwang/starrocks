---
displayed_sidebar: English
---

# 转换

## 描述

用于在 JSON 类型和 SQL 类型之间转换值。

## 语法

- 从 JSON 转换到 SQL

```Haskell
cast(json_expr AS sql_data_type)
```

- 从 SQL 转换到 JSON

```Haskell
cast(sql_expr AS JSON)
```

## 参数

- json_expr：表示你想转换为 SQL 值的 JSON 值的表达式。

- sql_data_type：你想将 JSON 值转换成的 SQL 数据类型。只支持以下数据类型：STRING、VARCHAR、CHAR、BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、DOUBLE 和 FLOAT。

- sql_expr：表示你想转换为 JSON 值的 SQL 值的表达式。此参数支持 sql_data_type 参数所支持的所有 SQL 数据类型。

## 返回值

- 如果使用 cast(json_expr AS sql_data_type) 语法，cast 函数将返回 sql_data_type 参数指定的 SQL 数据类型的值。

- 如果使用 cast(sql_expr AS JSON) 语法，cast 函数将返回一个 JSON 值。

## 使用说明

- 从 SQL 转换到 JSON

-   如果 SQL 值超出了 JSON 所支持的精度，cast 函数将返回 NULL 以避免发生算术溢出。

-   如果 SQL 值为 NULL，cast 函数不会将其转换为 NULL 的 JSON 值，返回值仍然是 NULL 的 SQL 值。

- 从 JSON 转换到 SQL

-   cast 函数只支持兼容的 JSON 和 SQL 数据类型之间的转换。例如，你可以将一个 JSON 字符串转换成 SQL 字符串。

-   cast 函数不支持不兼容的 JSON 和 SQL 数据类型之间的转换。例如，如果你尝试将一个 JSON 数字转换成 SQL 字符串，函数将返回 NULL。

-   如果发生算术溢出，cast 函数将返回 NULL 的 SQL 值。

-   如果你将一个 NULL 的 JSON 值转换成 SQL 值，函数将返回 NULL 的 SQL 值。

-   如果你将一个 JSON 字符串转换成 VARCHAR 值，函数返回的 VARCHAR 值将不会包含在双引号 (") 里。

## 示例

示例 1：将一个 JSON 值转换成 SQL 值。

```plaintext
-- Convert a JSON value to an INT value.
mysql> select cast(parse_json('{"a": 1}') -> 'a' as int);
+--------------------------------------------+
| CAST((parse_json('{"a": 1}')->'a') AS INT) |
+--------------------------------------------+
|                                          1 |
+--------------------------------------------+

-- Convert a JSON string to a VARCHAR value.
mysql> select cast(parse_json('"star"') as varchar);
+---------------------------------------+
| cast(parse_json('"star"') AS VARCHAR) |
+---------------------------------------+
| star                                  |
+---------------------------------------+

-- Convert a JSON object to a VARCHAR value.
mysql> select cast(parse_json('{"star": 1}') as varchar);
+--------------------------------------------+
| cast(parse_json('{"star": 1}') AS VARCHAR) |
+--------------------------------------------+
| {"star": 1}                                |
+--------------------------------------------+

-- Convert a JSON array to a VARCHAR value.

mysql> select cast(parse_json('[1,2,3]') as varchar);
+----------------------------------------+
| cast(parse_json('[1,2,3]') AS VARCHAR) |
+----------------------------------------+
| [1, 2, 3]                              |
+----------------------------------------+
```

示例 2：将一个 SQL 值转换成 JSON 值。

```plaintext
-- Convert an INT value to a JSON value.
mysql> select cast(1 as json);
+-----------------+
| cast(1 AS JSON) |
+-----------------+
| 1               |
+-----------------+

-- Convert a VARCHAR value to a JSON value.
mysql> select cast("star" as json);
+----------------------+
| cast('star' AS JSON) |
+----------------------+
| "star"               |
+----------------------+

-- Convert a BOOLEAN value to a JSON value.
mysql> select cast(true as json);
+--------------------+
| cast(TRUE AS JSON) |
+--------------------+
| true               |
+--------------------+
```
