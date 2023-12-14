---
displayed_sidebar: "Chinese"
---

# 取消导出

## 描述

取消给定的数据卸载任务。状态为 `CANCELLED` 或 `FINISHED` 的卸载任务无法取消。取消卸载任务是一个异步过程。您可以使用 [SHOW EXPORT](../data-manipulation/SHOW_EXPORT.md) 语句来检查卸载任务是否成功取消。如果 `State` 的值为 `CANCELLED`，则卸载任务已成功取消。

取消导出语句要求您在给定卸载任务所属的数据库上至少拥有以下权限之一：`SELECT_PRIV`、`LOAD_PRIV`、`ALTER_PRIV`、`CREATE_PRIV`、`DROP_PRIV` 和 `USAGE_PRIV`。有关权限描述的更多信息，请参见 [GRANT](../account-management/GRANT.md)。

## 语法

```SQL
CANCEL EXPORT
[FROM db_name]
WHERE QUERYID = "query_id"
```

## 参数

| **参数**    | **是否必需** | **描述**                                                     |
| ----------- | ------------ | ------------------------------------------------------------ |
| db_name     | 否           | 卸载任务所属的数据库的名称。如果未指定此参数，则将取消当前数据库中的卸载任务。 |
| query_id    | 是           | 卸载任务的查询 ID。您可以使用 [LAST_QUERY_ID()](../../sql-functions/utility-functions/last_query_id.md) 函数来获取该 ID。请注意，此函数仅返回最新的查询 ID。 |

## 示例

示例 1：取消当前数据库中查询 ID 为 `921d8f80-7c9d-11eb-9342-acde48001121` 的卸载任务。

```SQL
CANCEL EXPORT
WHERE queryid = "921d8f80-7c9d-11eb-9342-acde48001121";
```

示例 2：取消 `example_db` 数据库中查询 ID 为 `921d8f80-7c9d-11eb-9342-acde48001121` 的卸载任务。

```SQL
CANCEL EXPORT 
FROM example_db 
WHERE queryid = "921d8f80-7c9d-11eb-9342-acde48001122";
```