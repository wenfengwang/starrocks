---
displayed_sidebar: English
---

# last_query_id

## 描述

获取当前会话中最近执行的查询的 ID。

## 语法

```Haskell
VARCHAR last_query_id();
```

## 参数

无

## 返回值

返回一个 VARCHAR 类型的值。

## 示例

```Plain
mysql> select last_query_id();
+--------------------------------------+
| last_query_id()                      |
+--------------------------------------+
| 7c1d8d68-bbec-11ec-af65-00163e1e238f |
+--------------------------------------+
1 行在集合中 (0.00 秒)
```

## 关键字

LAST_QUERY_ID