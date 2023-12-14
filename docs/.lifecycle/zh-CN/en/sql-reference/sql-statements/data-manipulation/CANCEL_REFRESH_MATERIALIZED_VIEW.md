---
displayed_sidebar: "Chinese"
---

# 取消刷新物化视图

## 描述

取消异步物化视图的刷新任务。

## 语法

```SQL
CANCEL REFRESH MATERIALIZED VIEW [<database_name>.]<materialized_view_name>
```

## 参数

| **参数**                | **必需** | **描述**                                                     |
| ----------------------- | -------- | ------------------------------------------------------------ |
| database_name           | 否       | 物化视图所在数据库的名称。如果未指定此参数，则使用当前数据库。 |
| materialized_view_name  | 是       | 物化视图的名称。                                             |

## 示例

示例1：取消异步刷新物化视图`lo_mv1`的刷新任务。

```SQL
CANCEL REFRESH MATERIALIZED VIEW lo_mv1;
```