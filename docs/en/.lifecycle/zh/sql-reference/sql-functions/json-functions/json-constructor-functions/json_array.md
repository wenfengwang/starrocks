---
displayed_sidebar: English
---

# json_array

## 描述

将 SQL 数组的每个元素转换为 JSON 值，并返回包含这些 JSON 值的 JSON 数组。

## 语法

```Haskell
json_array(value, ...)
```

## 参数

`value`：SQL 数组中的一个元素。仅支持 `NULL` 值和以下数据类型：`STRING`、`VARCHAR`、`CHAR`、`JSON`、`TINYINT`、`SMALLINT`、`INT`、`BIGINT`、`LARGEINT`、`DOUBLE`、`FLOAT` 和 `BOOLEAN`。

## 返回值

返回一个 JSON 数组。

## 示例

示例 1：构建一个包含不同数据类型值的 JSON 数组。

```plaintext
mysql> SELECT json_array(1, true, 'starrocks', 1.1);

       -> [1, true, "starrocks", 1.1]
```

示例 2：构建一个空的 JSON 数组。

```plaintext
mysql> SELECT json_array();

       -> []
```